# page-aligned-paper-translation

一个 Claude Skill：把学术论文 PDF 翻译成中文（或其他语言），生成一份**与原文逐页对应**的新 PDF。

A Claude Skill that translates an academic paper PDF into Chinese (or another language) as a new PDF whose pages correspond 1:1 to the original.

## 特点

- **逐页对齐**：译本第 n 页恰好对应原文第 n 页的内容，跨页、跨栏的句子在原文断开的位置断开。
- **矢量图表**：图和表直接从原 PDF 矢量裁剪，放在原来的位置，只翻译图注和表注。
- **公式重排**：公式用 LaTeX 重新排版，保留原编号。
- **参考文献保留原文**。
- **译注**：发现原文的错误或未定义的符号时，可以加简短的译注。
- **先查许可证**：翻译前先检查论文的许可，并按许可类型决定能做什么（见下文）。

## 安装

把 `page-aligned-paper-translation/` 文件夹放进你的 skills 目录，或把该文件夹打包成 zip 后在 Claude 中上传为 skill。

## 使用

在对话中调用这个 skill，并提供论文 PDF。Claude 会先确认许可证，再和你商定审阅方式、译注规则和术语表，然后逐页翻译、编译并逐页对照检查。

运行环境需要：XeLaTeX、Noto Serif CJK SC 字体、TeX Gyre Termes 字体、poppler-utils（`pdftoppm`、`pdftotext`）、qpdf，以及 Python（Pillow、NumPy）。

## 版权与许可提醒

本 skill 只提供翻译**流程**，不包含任何论文或译文。翻译前请确认论文的许可：

- **CC BY / CC BY-SA**：允许翻译。
- **CC BY-NC-ND / BY-ND**：仅限个人非商业使用，译本不得分发。
- **出版商独占许可、保留所有权利或许可不明**：默认不做全文翻译，只提供学习笔记或术语表；只有使用者明确确认**仅供本人学习、不分享**时才做全文译本，且每页都要标注“仅限本人学习使用，不得分享”。

请不要上传或分享由受版权保护的论文生成的译本。

## License

本仓库中的 skill 文本按 [LICENSE](LICENSE) 中的条款发布。
