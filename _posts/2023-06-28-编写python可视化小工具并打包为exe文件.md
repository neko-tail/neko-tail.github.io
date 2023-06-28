---
tags: Python
category: Python
---

使用python编写小工具，为其添加可视化界面，并打包为exe可执行文件（主要是为了提供给他人使用），仅使用pyside6的情况下，exe文件大小为25M左右

<!--more-->

>   使用的库：
>
>   python环境[conda](https://docs.conda.io/projects/conda/en/stable/index.html)
>
>   打包[pyinstaller](https://pyinstaller.org/en/stable/)
>
>   可视化[pyside6](https://www.pythonguis.com/pyside6/)

### 使用conda创建虚拟环境

#### 为什么需要使用虚拟环境？

pyinstaller打包时会将当前python环境的所有依赖都打入可执行文件中，导致文件过于臃肿

所以需要一个仅包含程序所需要依赖的环境

#### 创建并使用虚拟环境

创建环境

```shell
# conda create 创建环境
# --name myenv 指定环境名
# python=3.10 指定python版本
# --no-default-packages 不安装默认的包
conda create --name myenv python=3.10 --no-default-packages
```

激活环境

```shell
# 激活环境
conda activate myenv
```

安装必要依赖，程序需要哪些依赖就装哪些依赖，这里我只安装了pyside6来可视化，pyinstaller来打包

```shell
conda install pyside6
conda install pyinstaller
```

### 使用pyinstaller打包

安装完依赖之后，使用以下命令打包

```shell
# -F 打包为单个exe文件，程序启动慢（因为启动时需要将文件解压到临时目录）
# -D 打包为文件夹，默认，程序启动快
# ./myapp.py 程序启动文件
# -w 启动时不显示命令行黑窗口
# --name myapp 指定可执行文件的名字
pyinstaller -F ./myapp.py -w --name myapp
```

执行命令后，输出可执行文件到`./dist`目录

### 使用pyside6制作可视化界面

参考代码如下，启动时打开一个窗口，具有选择文件控件，下拉框，按钮。点击按钮获取选择的文件路径，下拉框选项并处理，并打开处理结果所在的文件夹

```python
import os
import sys

from PySide6.QtCore import Qt, QSize
from PySide6.QtGui import QDragEnterEvent, QDropEvent, QFont
from PySide6.QtWidgets import QApplication, QMainWindow, QLabel, QVBoxLayout, QWidget, QComboBox, QPushButton, QFileDialog, QHBoxLayout, QSizePolicy, \
    QMessageBox

import my_process


class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        # 文件路径
        self.th_file_path = None
        # 设置窗口
        self.setWindowTitle("测试窗口")
        self.setFixedSize(QSize(400, 300))
        # 设置窗口字体大小
        font = QFont()
        font.setPointSize(12)
        label_font = QFont()
        label_font.setPointSize(10)
        self.setFont(font)

        # 创建拖拽区域标签
        self.drag_label = QLabel(self)
        self.drag_label.setAlignment(Qt.AlignCenter)
        self.drag_label.setSizePolicy(QSizePolicy.Expanding, QSizePolicy.Expanding)
        self.drag_label.setText("拖拽文件到这里")
        self.drag_label.setStyleSheet("QLabel { background-color: white; border: 1px solid gray; }")
        self.setAcceptDrops(True)

        # 创建用于显示文件路径的标签
        self.file_path_label = QLabel(self)
        self.file_path_label.setAlignment(Qt.AlignCenter)
        self.file_path_label.setFont(label_font)
        self.file_path_label.setWordWrap(True)
        self.file_path_label.setTextInteractionFlags(Qt.TextSelectableByMouse)
        # 调整标签的大小以适应文本
        self.file_path_label.adjustSize()

        # 创建下拉框
        self.combo_box = QComboBox(self)
        self.combo_box.addItem("option 1")
        self.combo_box.addItem("option 2")
        self.combo_box.addItem("option 3")
        self.combo_box.setFixedWidth(self.combo_box.sizeHint().width())

        # 创建按钮
        self.button = QPushButton("提交", self)
        self.button.clicked.connect(self.process_start)
        self.button.setFixedWidth(self.button.sizeHint().width())

        # 创建控件的水平布局
        cont_layout = QHBoxLayout()
        cont_layout.addWidget(self.combo_box)
        cont_layout.addWidget(self.button)

        # 创建垂直布局
        layout = QVBoxLayout()
        layout.addWidget(self.drag_label)
        layout.addWidget(self.file_path_label)
        layout.addSpacing(10)
        layout.addLayout(cont_layout)
        layout.addSpacing(10)

        # 创建主窗口的中心部件并设置布局
        central_widget = QWidget()
        central_widget.setLayout(layout)
        self.setCentralWidget(central_widget)

        # 连接点击事件到打开文件浏览器函数
        self.drag_label.mousePressEvent = self.open_file_dialog

    def dragEnterEvent(self, event: QDragEnterEvent):
        # 检查拖拽事件中的数据类型
        if event.mimeData().hasUrls():
            event.acceptProposedAction()

    def dropEvent(self, event: QDropEvent):
        # 获取拖拽的文件路径
        urls = event.mimeData().urls()
        file_paths = [url.toLocalFile() for url in urls]

        # 更新文件路径标签
        self.file_path_label.setText("\n".join(file_paths))
        self.th_file_path = file_paths[0]

    def open_file_dialog(self, event):
        # 打开文件对话框
        file_dialog = QFileDialog()
        file_path, _ = file_dialog.getOpenFileName(self, "选择文件")
        if file_path:
            # 更新文件路径标签
            self.file_path_label.setText(file_path)
            self.th_file_path = file_path

    def process_start(self):
        # 获取下拉框的选项
        selection = self.combo_box.currentText()

        # 开始处理
        if self.th_file_path:
            # 处理数据
            result_dest = my_process.process(option=selection, path=self.th_file_path)
            # 打开处理结果所在的目录
            os.startfile(os.path.abspath(result_dest))
        else:
            message_box = QMessageBox()
            message_box.setWindowTitle("提示")
            message_box.setText("未选择文件！")
            message_box.exec()


if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = MainWindow()
    window.show()
    sys.exit(app.exec())

```
