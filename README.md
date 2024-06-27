English | [简体中文](./README-zh_CN.md)

## 整体介绍

PDF文档中包含大量知识信息，然而提取高质量的PDF内容并非易事。为此，我们将PDF内容提取工作进行拆解：
- 布局检测：使用[LayoutLMv3](https://github.com/microsoft/unilm/tree/master/layoutlmv3)模型进行区域检测，如`图像`，`表格`,`标题`,`文本`等；
- 公式检测：使用[YOLOv8](https://github.com/ultralytics/ultralytics)进行公式检测，包含`行内公式`和`行间公式`；
- 公式识别：使用[UniMERNet](https://github.com/opendatalab/UniMERNet)进行公式识别；
- 光学字符识别：使用[PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR)进行文本识别；

> **注意：** *由于文档类型的多样性，现有开源的布局检测和公式检测很难处理多样性的PDF文档，为此我们内容采集多样性数据进行标注和训练，使得在各类文档上取得精准的检测效果，细节参考[布局检测](#layout-anchor)和[公式检测](#mfd-anchor)部分。对于公式识别，UniMERNet方法可以媲美商业软件，在各种类型公式识别上均匀很高的质量。对于OCR，我们采用PaddleOCR，对中英文OCR效果不错。*


PDF内容提取框架如下图所示

![](assets/demo/pipeline_v2.png)


<details>
  <summary>PDF-Extract-Kit输出格式</summary>

```Bash
{
    "layout_dets": [    # 页中的元素
        {
            "category_id": 0, # 类别编号， 0~9，13~15
            "poly": [
                136.0, # 坐标为图片坐标，需要转换回pdf坐标, 顺序是 左上-右上-右下-左下的x,y坐标
                781.0,
                340.0,
                781.0,
                340.0,
                806.0,
                136.0,
                806.0
            ],
            "score": 0.69,   # 置信度
            "latex": ''      # 公式识别的结果，只有13,14有内容，其他为空，另外15是ocr的结果，这个key会换成text
        },
        ...
    ],
    "page_info": {         # 页信息：提取bbox时的分辨率大小，如果有缩放可以基于该信息进行对齐
        "page_no": 0,      # 页数
        "height": 1684,    # 页高
        "width": 1200      # 页宽
    }
}
```

其中category_id包含的类型如下：

```
{0: 'title',              # 标题
 1: 'plain text',         # 文本
 2: 'abandon',            # 包括页眉页脚页码和页面注释
 3: 'figure',             # 图片
 4: 'figure_caption',     # 图片描述
 5: 'table',              # 表格
 6: 'table_caption',      # 表格描述
 7: 'table_footnote',     # 表格注释
 8: 'isolate_formula',    # 行间公式（这个是layout的行间公式，优先级低于14）
 9: 'formula_caption',    # 行间公式的标号

 13: 'inline_formula',    # 行内公式
 14: 'isolated_formula',  # 行间公式
 15: 'ocr_text'}              # ocr识别结果
```
</details>


## 效果展示

结合多样性PDF文档标注，我们训练了鲁棒的`布局检测`和`公式检测`模型。在论文、教材、研报、财报等多样性的PDF文档上，我们的pipeline都能得到准确的提取结果，对于扫描模糊、水印等情况也有较高鲁棒性。


![](assets/demo/example1.png)

![](assets/demo/example2.png)

## 评测指标

现有开源模型多基于Arxiv论文类型数据进行训练，面对多样性的PDF文档，提前质量远不能达到实用需求。相比之下，我们的模型经过多样化数据训练，可以适应各种类型文档提取。

<span id="layout-anchor"></span>
### 布局检测
TODO

<span id="mfd-anchor"></span>
### 公式检测
TODO

### 公式识别


## 使用教程

### 环境安装

```
conda create -n pipeline python=3.10

pip install unimernet

pip install -r requirements.txt

pip install --extra-index-url https://miropsota.github.io/torch_packages_builder detectron2==0.6+pt2.2.2cu121
```

安装完环境后，可能会遇到一些版本冲突导致版本变更，如果遇到了版本相关的报错，可以尝试下面的命令重新安装指定版本的库。

```
pip uninstall PyMuPDF

pip install PyMuPDF==1.20.2

pip install pillow==8.4.0
```

除了版本冲突外，可能还会遇到torch无法调用的错误，可以先把下面的库卸载，然后重新安装cuda12和cudnn。

```
pip uninstall nvidia-cusparse-cu12
```

### 参考[模型下载](models/README.md)下载所需模型权重



## 运行提取脚本

```bash 
python pdf_extract.py --pdf data/pdfs/ocr_1.pdf
```

相关参数解释：
- --pdf 待处理的pdf文件，如果传入一个文件夹，则会处理文件夹下的所有pdf文件。
- --output 处理结果保存的路径，默认是"output"
- --vis 是否对结果可视化，是则会把检测的结果可视化出来，主要是检测框和类别
- --render 是否把识别得的结果渲染出来，包括公式的latex代码，以及普通文本，都会渲染出来放在检测框中。注意：此过程非常耗时，另外也需要提前安装`xelatex`和`imagemagic`。




## Acknowledgement

   - [LayoutLMv3](https://github.com/microsoft/unilm/tree/master/layoutlmv3): 布局检测模型
   - [UniMERNet](https://github.com/opendatalab/UniMERNet): 公式识别模型
   - [YOLOv8](https://github.com/ultralytics/ultralytics): 公式检测模型
   - [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR): OCR模型