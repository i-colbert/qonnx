# QONNX: Arbitrary-Precision Quantized Neural Networks in ONNX

[![ReadTheDocs](https://readthedocs.org/projects/finn-base/badge/?version=latest&style=plastic)](http://finn-base.readthedocs.io/)
[![GitHub Discussions](https://img.shields.io/github/discussions/fastmachinelearning/qonnx)](https://github.com/fastmachinelearning/qonnx/discussions)
![Tests](https://github.com/fastmachinelearning/qonnx/actions/workflows/test.yml/badge.svg)


<img align="left" src="https://xilinx.github.io/finn/img/TFC_1W2A.onnx.png" alt="QONNX example" style="margin-right: 20px" width="200"/>


QONNX (Quantized ONNX) introduces three new custom operators -- [`Quant`](docs/qonnx-custom-ops/quant_op.md), [`BipolarQuant`](docs/qonnx-custom-ops/bipolar_quant_op.md), and [`Trunc`](docs/qonnx-custom-ops/trunc_op.md) -- in order to represent arbitrary-precision uniform quantization in ONNX. This enables:
* Representation of binary, ternary, 3-bit, 4-bit, 6-bit or any other quantization.
* Quantization is an operator itself, and can be applied to any parameter or layer input.
* Flexible choices for scaling factor and zero-point granularity.
* Quantized values are carried using standard `float` datatypes to remain ONNX protobuf-compatible.

This repository contains a set of Python utilities to work with QONNX models, including but not limited to:
* executing QONNX models for (slow) functional verification
* shape inference, constant folding and other basic optimizations
* summarizing the inference cost of a QONNX model in terms of mixed-precision MACs, parameter and activation volume
* Python infrastructure for writing transformations and defining executable, shape-inferencable custom ops
* (experimental) data layout conversion from standard ONNX NCHW to custom QONNX NHWC ops

## Quickstart

### Operator definitions

* [Quant](docs/qonnx-custom-ops/quant_op.md) for 2-to-arbitrary-bit quantization, with scaling and zero-point
* [BipolarQuant](docs/qonnx-custom-ops/bipolar_quant_op.md)  for 1-bit (bipolar) quantization, with scaling and zero-point
* [Trunc](docs/qonnx-custom-ops/trunc_op.md) for truncating to a specified number of bits, with scaling and zero-point

### Installation

`pip install qonnx`

### Export, Import and Model Zoo

The following quantization-aware training (QAT) frameworks support exporting to QONNX:

* [Brevitas](https://github.com/Xilinx/brevitas)
* [QKeras](https://github.com/google/qkeras) (beta, see [this PR](https://github.com/fastmachinelearning/qonnx/pull/7))
* HAWQ (coming soon)
* [<your NN quantization framework here? please get in touch!>](https://github.com/fastmachinelearning/qonnx/discussions)

The following NN inference frameworks support importing QONNX models for deployment:

* [FINN](https://github.com/Xilinx/finn) (FPGA dataflow-style)
* [hls4ml](https://github.com/fastmachinelearning/hls4ml) (FPGA dataflow-style)
* [<your NN deployment framework here? please get in touch!>](https://github.com/fastmachinelearning/qonnx/discussions)

Head to the [QONNX model zoo](https://github.com/fastmachinelearning/QONNX_model_zoo) to download pre-trained QONNX models on various datasets.

### Model Visualization

We recommend [Netron](https://netron.app/) for visualizing QONNX models.

### Executing ONNX graph with QONNX custom nodes

Using the `qonnx-exec` command line utility, with top-level inputs supplied from `in0.npy` and `in1.npy`:

`qonnx-exec my-qonnx-model.onnx in0.npy in1.npy`

Using the Python API:

```
from qonnx.core.modelwrapper import ModelWrapper
from qonnx.core.onnx_exec import execute_onnx

model = ModelWrapper("my-qonnx-model.onnx")
idict = {"in0" : np.load("in0.npy), "in1" : np.load("in1.npy")}
odict = execute_onnx(idict)
```

### Calculate inference cost for QONNX model

Using the `qonnx-inference-cost` command line utility:

`qonnx-inference-cost CNV_2W2A.onnx`

Which will print a inference cost dictionary like the following:

```
Inference cost for CNV_2W2A.onnx
{
  "discount_sparsity": true,    # discount MAC counts by layer sparsity (disregard zero-valued MACs)
  # mem_o_X: number of layer outputs with datatype X
  "mem_o_FLOAT32": 57600.0,     # number of FLOAT32 output elements
  "mem_o_INT32": 85002.0,       # number of INT32 output elements
  # mem_o_X: number of layer parameters (weights) with datatype X
  "mem_w_INT2": 1144512.0,      # number of INT2 parameters (weights)
  # op_mac_X_Y: number of MAC operations, datatype X by datatype Y
  "op_mac_FLOAT32_INT2": 1555200.0, # number of float32 x int2 MACs
  "op_mac_INT2_INT2": 57906176.0,   # number of int2 x int2 MACs
  "total_bops": 331157504.0,        # total number of MACs normalized to bit-ops (BOPS)
  "total_mem_o_bits": 4563264.0,    # total number of bits for layer outputs
  "total_mem_w_bits": 2289024.0,    # total number of bits for layer parameters
  "unsupported": "set()"
}
```

You can read more about the BOPS metric in [this paper](https://www.frontiersin.org/articles/10.3389/frai.2021.676564/full), Section 4.2 Bit Operations.

### Development

Install in editable mode in a venv:

```
git clone https://github.com/fastmachinelearning/qonnx
cd qonnx
virtualenv -p python3.7 venv
source venv/bin/activate
pip install -e .[testing]
```

Run entire test suite, parallelized across CPU cores:
```
pytest -n auto --verbose
```

Run a particular test and fall into pdb if it fails:
```
pytest --pdb -k "test_extend_partition.py::test_extend_partition[extend_id1-2]"
```

## Why QONNX?

The QONNX representation has several advantages compared to other alternatives, as summarized in the table below.
These include a compact but flexible, single-node quantization representation that avoids operator duplication
and can support arbitrary precision up to the container datatype limit.

<img align="left" src="docs/qonnx-comparison.png" alt="QONNX comparison table" style="margin-right: 20px" />

## Community

The QONNX efforts were started by the FINN and hls4ml communities working together to create a common, arbitrary-precision representation that both frameworks could ingest. However, QONNX aims to build an open-source community for practitioners and researchers working with mixed-precision quantized neural networks by providing useful tools and a [discussion forum](https://github.com/fastmachinelearning/qonnx/discussions).

<div>
<img src=https://raw.githubusercontent.com/Xilinx/finn/github-pages/docs/img/finn-logo.png height=100/>
<img src="https://fastmachinelearning.github.io/hls4ml/img/logo.jpg" alt="hls4ml" height="128"/>
</div>

## Resources

You can read more about QONNX in [this paper](https://arxiv.org/abs/2206.07527). If you find QONNX useful in your work, please consider a citation:

```
@article{pappalardo2022qonnx,
  title={QONNX: Representing Arbitrary-Precision Quantized Neural Networks},
  author={Pappalardo, Alessandro and Umuroglu, Yaman and Blott, Michaela and Mitrevski, Jovan and Hawks, Ben and Tran, Nhan and Loncar, Vladimir and Summers, Sioni and Borras, Hendrik and Muhizi, Jules and others},
  journal={arXiv preprint arXiv:2206.07527},
  year={2022}
}
```