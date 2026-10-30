# On-device speech recognition

**A wav2vec2 phoneme model running on an iPhone, with no server in the loop.**

`Python` `PyTorch` `coremltools` `Core ML` `iOS`

---

## The problem

A speech-therapy product scores a child's pronunciation from a short recording. Three
constraints arrived together:

1. **Latency.** Feedback has to land well under a second. A round trip to a server does not.
2. **Privacy.** The input is a recording of a child. It should not leave the device.
3. **Variable length.** Children speak for one second or ten. The model cannot assume a
   fixed window.

That rules out the obvious architecture — audio → network → model → network → result — and
moves the model onto the phone. On-device inference then adds its own constraints: the
package has to fit in an app bundle, it has to load fast enough that the first word is not
delayed, and it has to accept whatever length the user actually speaks.

## The interesting part

The standard route for PyTorch → Core ML is PyTorch → ONNX → Core ML. It failed here, four
times, and every error message pointed at Core ML when the problem was the frontend:

- `optimum` exported an ONNX graph with a documented numerical drift (`max diff =
  6.1e-05`) before Core ML was involved. For a CTC model whose logits feed an argmax, that
  is harmless in most frames and not harmless at a decision boundary.
- `coremltools.convert(onnx.load(...))` could not identify the source framework.
- ONNX marked the sequence axis unbounded, and `mlprogram` rejects an infinite upper bound.
- The converter then rejected the input object itself, asking for a TorchScript object
  rather than a `PosixPath`.

The working route traces the PyTorch module directly:

```python
model = AutoModelForCTC.from_pretrained(model_id, torchscript=True)
model.eval()
traced = torch.jit.trace(model, torch.rand(1, 16_000))

coreml_model = ct.convert(
    traced,
    inputs=[ct.TensorType(
        name="input_values",
        shape=(1, ct.RangeDim(lower_bound=1, upper_bound=480_000, default=16_000)),
    )],
    convert_to="mlprogram",
    minimum_deployment_target=ct.target.iOS15,
)
```

Three details make it work:

- **Trace, don't export.** `torchscript=True` makes the model return tuples instead of a
  dataclass, which the tracer can flatten. The output name is generated (`var_1732` in one
  build), so it is read from the spec rather than hard-coded.
- **Tracing and shape declaration are separate.** `torch.jit.trace` fixes the graph at one
  second. `ct.RangeDim(1, 480_000, 16_000)` declares the range the model accepts at
  runtime. Both are required.
- **`model.eval()` is not optional.** A traced dropout is a different model, and it fails
  silently.

## The precision decision

Three packages, same input contract, so they can be swapped behind one Swift protocol.

> These numbers come from the **public rebuild** (`model-on-device`, a public checkpoint),
> not from the company build. The company build shipped FP32 and FP16; the rebuild is
> where the INT8 comparison happens.

| Package | Size | Cold load | 1 s | 5 s | 10 s | 30 s | Token agreement |
|---|---:|---:|---:|---:|---:|---:|---:|
| FP32 | 1262 MB | 5.4 s | 24 ms | 83 ms | 166 ms | 534 ms | 100.0% |
| FP16 | 631 MB | 64 s | 13 ms | 65 ms | 127 ms | 402 ms | 100.0% |
| INT8 | 318 MB | 4.0 s | 24 ms | 83 ms | 166 ms | 536 ms | 99.6% |

Device: Apple M4, ten runs per length, median.

The row that matters is not the fastest one. It is the smallest package that still agrees
with PyTorch on the tokens, because a pronunciation score that is 0.4% wrong is worse than
a slow one that is right.

Two rows tell the story:

- **FP16 is the shipping candidate.** Roughly half of FP32's latency, same token
  agreement, half the size. Its cold load costs about a minute because Core ML compiles the
  half-precision program in-process on first use. A runtime download hides that behind a
  progress bar; a bundled model shows it. The real fix is shipping the compiled
  `.mlmodelc`.
- **INT8 is weight-only.** Activations stay in float, because coremltools' activation
  quantizer raises a scale-dtype mismatch on this graph in both 8.3 and 9.0. The size win
  is real; the latency win is not the one "INT8" usually implies.

## What I would do next

- Pre-compile the FP16 package to `.mlmodelc` at build time.
- Ship a small but real calibration and eval set, so INT8's accuracy cost is measured on
  speech instead of noise.
- Add a Swift sample app with the microphone and a latency readout, so the benchmark
  matches the experience.
- Compare against `SFSpeechRecognizer` for the same task, because "we need a custom model"
  is a claim that should be tested.

---

The code lives in [model-on-device](https://github.com/limanup/model-on-device), a clean
rebuild around a public checkpoint
(`vitouphy/wav2vec2-xls-r-300m-timit-phoneme`). No customer data, no product code, no
company paths.
