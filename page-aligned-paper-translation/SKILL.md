---
name: "page-aligned-paper-translation"
description: "Translate an academic paper PDF into Chinese (or another language) as a new PDF whose pages correspond 1:1 to the original, with figures/tables cropped as vectors from the original."
---

# Page-aligned paper translation

Goal: a translated PDF where page n holds exactly the content of original page n. Figures and tables are vector crops of the original PDF. Equations are retyped in LaTeX with the original numbers. Reference pages are kept as the original pages.

## 0. Before any work

### 0.1 Check the license
Find the license on the arXiv abs page ("view license") or the publisher page. For journal papers, also check page 1 of the PDF and the last page for a copyright line (e.g. "© The Author(s), under exclusive licence to Springer…"). If you cannot read it, ask the user to check it.

- **CC BY / CC BY-SA:** translating is allowed.
- **CC BY-NC-ND 4.0 (or BY-ND):** private adaptations for non-commercial use are allowed, but they must not be shared. Proceed only for personal use. Stamp page 1 with "个人学习用中文译本，依原文许可 … 不得分发" plus the arXiv id.
- **Publisher exclusive licence (e.g. Springer "under exclusive licence"), arXiv default non-exclusive license, all-rights-reserved, or unknown:**
  - Tell the user the license type first, and offer the alternatives: page-aligned study notes (per-paragraph explanations with short quotes), or running a local tool (PDFMathTranslate, 沉浸式翻译) on their own copy with a glossary from you.
  - If the user explicitly confirms the translation is **only for their own personal study** and will not be shared, uploaded or used for anything else, you may produce the full translation. In that case:
    - Stamp **every page** (top margin, small red text): "个人学习用中文译本——仅限本人学习使用，不得分享、传播或用于其他任何用途。原文：<作者> et al., <期刊/会议> (<年份>)，DOI <doi>，© <版权方>（独占许可）。"
    - Keep the original copyright line on page 1 and the publisher's notice on the last page untranslated and visible.
    - In the final reply, remind the user that the paper is not CC-licensed (name the publisher and license type) and that the translation must stay with them only: no sharing, no uploading.
  - Without that confirmation, do not produce a full translation.
- Never publish the translation as an Artifact or anywhere shareable. Deliver it as a file into the user's own folder only.

### 0.2 Agree on rules with the user
Ask once, then start:
1. Review granularity: per paragraph, per page, or all at the end.
2. Errors in the original: translate faithfully, or add short translator notes (译注) as footnotes.
3. Glossary. Propose candidates, for example: domain → 领域, zero-shot → 零样本, ridge regression → 岭回归. Keep metric names, method names and dataset names in English.

## 1. Setup

Prepare the source:
- `qpdf orig_raw.pdf orig.pdf` repairs xref errors.
- `pdftoppm -r 288 -png orig.pdf hi/p` renders hi-res pages for coordinate work.
- `pdftoppm -r 110 -png orig.pdf pg` renders pages for reading.
- `pdftotext -bbox-layout orig.pdf bbox.html` gives the coordinates of every text block. Units are pt, with the origin at the top-left.

Typeset with XeLaTeX:
- Fonts: Noto Serif CJK SC and TeX Gyre Termes (Times-like).
- If xeCJK/ctex are missing and CTAN is blocked, do NOT compile packages downloaded from GitHub. Use the minimal switcher below instead.
- Do not use ucharclasses with \begingroup/\endgroup transitions; they break.

```latex
% cjksetup.tex
\newfontfamily\cjkfont[NFSSFamily=cjk,BoldFont={Noto Serif CJK SC Bold},ItalicFont={Noto Serif CJK SC},BoldItalicFont={Noto Serif CJK SC Bold}]{Noto Serif CJK SC}
\makeatletter
\def\cjk@fam{cjk}\def\cjk@saved{\rmdefault}
\def\tocjk{\ifx\f@family\cjk@fam\else\edef\cjk@saved{\f@family}\fontfamily{cjk}\selectfont\fi}
\def\fromcjk{\ifx\f@family\cjk@fam\fontfamily{\cjk@saved}\selectfont\fi}
\newXeTeXintercharclass\cjkclass \newcount\cjk@c
\def\cjk@range#1#2{\cjk@c=#1\relax\loop\XeTeXcharclass\cjk@c=\cjkclass\advance\cjk@c\@ne\ifnum\cjk@c<\numexpr#2+1\relax\repeat}
\cjk@range{"2E80}{"2FFF}\cjk@range{"3000}{"303F}\cjk@range{"3040}{"9FFF}
\cjk@range{"F900}{"FAFF}\cjk@range{"FE30}{"FE4F}\cjk@range{"FF00}{"FFEF}
\XeTeXinterchartoks 0 \cjkclass = {\tocjk}\XeTeXinterchartoks 4095 \cjkclass = {\tocjk}
\XeTeXinterchartoks \cjkclass 0 = {\fromcjk}\XeTeXinterchartoks \cjkclass 4095 = {\fromcjk}
\makeatother
\XeTeXinterchartokenstate=1 \XeTeXlinebreaklocale "zh" \XeTeXlinebreakskip=0pt plus 1pt
```

## 2. Main file

Copy the original geometry. Measure it from bbox.html: page size, the left x of the text, the top y, the text height, and the page-number baseline. The NeurIPS template is: letter 612×792bp, `textwidth=396bp, textheight=650bp, left=108bp, top=72bp, footskip=30bp`.

Key preamble pieces:
- `\newcommand{\cnsize}{\fontsize{10}{12.8}\selectfont}`. 10pt CJK with 12.8pt leading takes about the same height as the English original.
- `\setlength{\parindent}{0pt}`, `\setlength{\parskip}{5pt plus 1pt}`, `\raggedbottom`.
- `\newcommand{\origfig}[2]{\includegraphics[page=#1,viewport=#2,clip]{orig.pdf}}` takes a viewport in bp with the origin at the bottom-left.
- `\newcommand{\pg}[1]{\setcounter{footnote}{0}\input{#1}\clearpage}`, used as `\pg{p01} … \pg{pNN}`.
- Use `\renewcommand{\thefootnote}{\fnsymbol{footnote}}` and `\newcommand{\yz}[1]{\footnote{\textbf{译注：}#1}}` for translator notes.
- Reference pages: `\includepdf[pages=a-b,pagecommand={\thispagestyle{empty}}]{orig.pdf}`. The original page numbers are kept.
- Sections: write manual numbered headings (`\sect{3}{…}`, `\subsect{3.1}{…}`) so the numbering matches the original.
- Equations: `\begin{equation}\tag{k} … \end{equation}` with the original numbers.
- Bold math macros: \bW, \bPhi, and so on.

## 3. Figure/table coordinates

`bbox.py page x0 y0 x1 y1` returns the tight non-white bbox inside a region (pt, top-left origin) and prints the viewport string `llx lly urx ury`, where lly = 792 − ymax.

```python
import sys; from PIL import Image; import numpy as np
p=int(sys.argv[1]); x0,y0,x1,y1=map(float,sys.argv[2:6]); s=288/72
im=np.array(Image.open(f'hi/p-{p:02d}.png').convert('L'))
sub=im[int(y0*s):int(y1*s),int(x0*s):int(x1*s)]; ys,xs=np.where(sub<245)
bx0=x0+xs.min()/s; bx1=x0+(xs.max()+1)/s; by0=y0+ys.min()/s; by1=y0+(ys.max()+1)/s
print(f'viewport={bx0-1:.1f} {792-by1-1:.1f} {bx1+1:.1f} {792-by0+1:.1f}')
```

- Pick the region from the caption positions in bbox.html: a figure ends just above its caption.
- Exclude the caption from the crop; it gets translated.
- When text wraps beside a figure, narrow the x-range to separate the figure from the text.
- Crop tables whole, without the caption. Rotated headers and bold numbers then stay exact.
- Translate subfigure captions that sit outside the crop.

## 4. Translate page by page

- Write one file per original page (`p01.tex`, …). Read each page from its rendered **image**, not from pdftotext output, because extracted math is garbled.
- Translate paragraph by paragraph, faithfully, using the agreed glossary. Keep citations [n], equation references and "Fig./Tab." numbers. Example class labels stay in English italics.
- If a sentence crosses the page boundary in the original, split the translation at the same logical point. End page n with the first half and start page n+1 with the rest.
- Place each figure at the same position as on the original page: top, middle, side-by-side minipages, or `wrapfigure`.
- Page 1:
  - Translated title, with the original title in small italics underneath.
  - Authors and affiliations as in the original.
  - Footnotes and the venue line at the bottom via `\vfill`.
  - The arXiv side stamp re-added with `\AddToShipoutPictureFG*` and `\put(x,y){\origfig{1}{…}}`.
  - The license notice in the top margin (on every page for publisher-licensed papers, see 0.1).
- Algorithms: use a simple rule-bounded box (`parskip 0`, `\nointerlineskip` around the rules) with manual indentation per level. Keep the for/if/end keywords in English bold.

## 5. Fit and verify after every compile

- Compile with `xelatex` and check `pdfinfo` Pages == the original page count.
- Print the first line of each output page with `pdftotext -f i -l i`, then locate the page that overflowed. Text inside included figure pages is hidden noise, so ignore it.
- Fix overflow on that page only, in this order:
  1. Reduce `\vspace` around titles, figures and algorithm rules.
  2. Use `\centering` instead of a `center` environment.
  3. Put short footnotes on one line, or move notices into the margin.
  4. Reduce the local leading slightly (no lower than about 12pt for 10pt CJK).
- Never cut content to make it fit.
- Make side-by-side PNGs (original | translation) for every page and inspect each one. Check figure placement, equations, page numbers and the footnotes.

## 6. Deliver

- Copy the PDF into the user's folder, for example `<paper>_中文译本.pdf`. Do not create an artifact.
- In the reply give:
  - the location and page count;
  - the glossary used;
  - the list of translator notes with page numbers;
  - honest caveats.
- Typical caveats:
  - Vector figure crops embed the hidden original page text, so Ctrl+F may hit invisible English.
  - Algorithm keywords are kept in English.
  - Punctuation spacing is slightly looser without xeCJK.
- Remind the user of the license limits, for example no sharing for ND licenses. For a publisher exclusive licence or all-rights-reserved paper translated for personal study (0.1), say plainly: "这篇论文是 <出版商> 独占许可，不是 CC 许可。请把这份译本只留给自己用，不要分享或上传到任何地方。"
