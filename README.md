# Marker

Marker 能够快速且准确地将 PDF 转换为 Markdown。

- 支持广泛的文档类型（针对书籍和科学论文进行了优化）
- 支持所有语言
- 移除页眉/页脚/其他杂项
- 格式化表格和代码块
- 提取并保存图像与 Markdown 一起
- 将大多数方程式转换为 LaTeX
- 在 GPU、CPU 或 MPS 上运行

## 工作原理

Marker 是一系列深度学习模型的管道：

- 提取文本，如有必要进行 OCR（启发式， [surya](https://github.com/VikParuchuri/surya)，tesseract）
- 检测页面布局并找到阅读顺序（[surya](https://github.com/VikParuchuri/surya)）
- 清理和格式化每个块（启发式，[texify](https://github.com/VikParuchuri/texify)）
- 合并块并后处理完整文本（启发式，[pdf_postprocessor](https://huggingface.co/vikp/pdf_postprocessor_t5)）

它只在必要时使用模型，从而提高速度和准确性。

## 示例

| PDF                                                                   | 类型        | Marker                                                                                                 | Nougat                                                                                                 |
|-----------------------------------------------------------------------|-------------|--------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| [Think Python](https://greenteapress.com/thinkpython/thinkpython.pdf) | 教科书      | [查看](https://github.com/VikParuchuri/marker/blob/master/data/examples/marker/thinkpython.md)         | [查看](https://github.com/VikParuchuri/marker/blob/master/data/examples/nougat/thinkpython.md)         |
| [Think OS](https://greenteapress.com/thinkos/thinkos.pdf)             | 教科书      | [查看](https://github.com/VikParuchuri/marker/blob/master/data/examples/marker/thinkos.md)             | [查看](https://github.com/VikParuchuri/marker/blob/master/data/examples/nougat/thinkos.md)             |
| [Switch Transformers](https://arxiv.org/pdf/2101.03961.pdf)           | arXiv 论文  | [查看](https://github.com/VikParuchuri/marker/blob/master/data/examples/marker/switch_transformers.md) | [查看](https://github.com/VikParuchuri/marker/blob/master/data/examples/nougat/switch_transformers.md) |
| [Multi-column CNN](https://arxiv.org/pdf/1804.07821.pdf)              | arXiv 论文  | [查看](https://github.com/VikParuchuri/marker/blob/master/data/examples/marker/multicolcnn.md)         | [查看](https://github.com/VikParuchuri/marker/blob/master/data/examples/nougat/multicolcnn.md)         |

## 性能

![基准总体](data/images/overall.png)

以上结果是使用 marker 和 nougat 设置，因此它们在 A6000 上各自消耗约 4GB 的 VRAM。

有关详细的速度和准确性基准，以及如何运行您自己的基准的说明，请参见 [下面](#benchmarks)。

# 商业使用

我希望 Marker 尽可能广泛地可用，同时仍能为我的开发/培训成本提供资金。 研究和个人使用总是可以的，但商业使用有一些限制。

模型的权重被授权为 `cc-by-nc-sa-4.0`，但我将对最近 12 个月内总收入低于 500 万美元 AND 终生 VC/天使融资低于 500 万美元的任何组织免除该限制。 您也不得与 [Datalab API](https://www.datalab.to/) 竞争。 如果您想移除 GPL 许可证要求（双重许可）和/或在超过收入限制的情况下商业使用权重，请查看 [这里](https://www.datalab.to)。

# 托管 API

Marker 的托管 API 可在 [这里](https://www.datalab.to/) 使用：

- 支持 PDF、Word 文档和 PowerPoint
- 价格为领先云竞争对手的四分之一
- 高正常运行时间（99.99%）、质量和速度（每页 0.25 秒，适用于 50 页文档）

# 社区

[Discord](https://discord.gg//KuZwXNGnfH) 是我们讨论未来发展的地方。

# 限制

PDF 是一种棘手的格式，因此 Marker 并不总是能完美工作。以下是一些已知的限制，正在进行改进：

- Marker 不会将 100% 的方程式转换为 LaTeX。这是因为它必须先检测再转换。
- 表格并不总是 100% 正确格式化 - 文本可能在错误的列中。
- 空白和缩进并不总是得到尊重。
- 并非所有行/跨度都将正确连接。
- 这对于不需要大量 OCR 的数字 PDF 最有效。它经过速度优化，并使用有限的 OCR 来修复错误。

# 安装

您需要 Python 3.9+ 和 PyTorch。如果您不使用 Mac 或 GPU 机器，可能需要先安装 CPU 版本的 torch。有关详细信息，请参见 [这里](https://pytorch.org/get-started/locally/)。

使用以下命令安装：

```shell
pip install marker-pdf
```

## 可选：OCRMyPDF

仅在您希望将可选的 `ocrmypdf` 作为 OCR 后端时需要。请注意，`ocrmypdf` 包含 Ghostscript，一个 AGPL 依赖项，但通过 CLI 调用，因此不会触发许可证条款。

请参见 [这里](docs/install_ocrmypdf.md) 的说明。

# 使用

首先，进行一些配置：

- 检查 `marker/settings.py` 中的设置。您可以通过环境变量覆盖任何设置。
- 您的 torch 设备将被自动检测，但您可以覆盖此设置。例如，`TORCH_DEVICE=cuda`。
- 默认情况下，Marker 将使用 `surya` 进行 OCR。Surya 在 CPU 上速度较慢，但比 tesseract 更准确。它还不需要您指定文档中的语言。如果您想要更快的 OCR，请将 `OCR_ENGINE` 设置为 `ocrmypdf`。这也需要外部依赖项（见上文）。如果您根本不想要 OCR，请将 `OCR_ENGINE` 设置为 `None`。
- 一些 PDF，即使是数字版，文本质量也很差。如果您发现 Marker 的输出有问题，请将 `OCR_ALL_PAGES=true` 设置为强制进行 OCR。

## 交互式应用

我包含了一个 Streamlit 应用程序，允许您以一些基本选项与 Marker 进行交互。运行它：

```shell
pip install streamlit
marker_gui
```

## 转换单个文件

```shell
marker_single /path/to/file.pdf /path/to/output/folder --batch_multiplier 2 --max_pages 10 
```

- `--batch_multiplier` 是指在您有额外 VRAM 时默认批大小的倍数。更高的数字将占用更多 VRAM，但处理更快。默认设置为 2。默认批大小将消耗约 3GB 的 VRAM。
- `--max_pages` 是要处理的最大页面数。省略此项以转换整个文档。
- `--start_page` 是要开始的页面（默认值为 None，将从第一页开始）。
- `--langs` 是文档中语言的可选逗号分隔列表，用于 OCR。默认情况下是可选的，如果您使用 tesseract，则为必需的。

surya OCR 支持的语言列表 [在这里](https://github.com/VikParuchuri/surya/blob/master/surya/languages.py)。如果您需要更多语言，可以使用任何由 [Tesseract](https://tesseract-ocr.github.io/tessdoc/Data-Files#data-files-for-version-400-november-29-2016) 支持的语言（前提是将 `OCR_ENGINE` 设置为 `ocrmypdf`）。如果您不需要 OCR，Marker 可以处理任何语言。

## 转换多个文件

```shell
marker /path/to/input/folder /path/to/output/folder --workers 4 --max 10
```

- `--workers` 是同时转换的 PDF 数量。默认设置为 1，但您可以增加它以提高吞吐量，代价是更多的 CPU/GPU 使用。Marker 在峰值时每个工作线程将使用 5GB 的 VRAM，平均使用 3.5GB。
- `--max` 是要转换的最大 PDF 数量。省略此项以转换文件夹中的所有 PDF。
- `--min_length` 是提取的 PDF 中需要的最小字符数，才能考虑进行处理。如果您处理大量 PDF，我建议设置此项，以避免 OCR 主要是图像的 PDF（这会减慢一切）。
- `--metadata_file` 是可选路径，指向包含 PDF 元数据的 JSON 文件。如果提供，将用于为每个 PDF 设置语言。对于 surya（默认），设置语言是可选的，但对于 tesseract
