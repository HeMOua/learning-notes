# openGL模块

## 准备

新建一个`render`子模块，并创建一个新类`QYUVOpenGLWidget`，如下

qyuvopenglwidget.h

```c++
#ifndef QYUVOPENGLWIDGET_H
#define QYUVOPENGLWIDGET_H

#include <QOpenGLWidget>
#include <QOpenGLFunctions>

class QYUVOpenGLWidget : public QOpenGLWidget, protected QOpenGLFunctions
{
    Q_OBJECT
public:
    QYUVOpenGLWidget(QWidget *parent = nullptr);

protected:
    virtual void initializeGL();
    virtual void resizeGL(int w, int h);
    virtual void paintGL();

signals:

};

#endif // QYUVOPENGLWIDGET_H
```

要实现openGL绘图，先要这个类继承`QOpenGLWidget`以及`QOpenGLFunctions`，并且还要实现三个虚函数

+ `virtual void initializeGL();`
+ `virtual void resizeGL(int w, int h);`
+ `virtual void paintGL();`

qyuvopenglwidget.cpp

```c++
#include "qyuvopenglwidget.h"

QYUVOpenGLWidget::QYUVOpenGLWidget(QWidget *parent)
    : QOpenGLWidget(parent)
{

}

void QYUVOpenGLWidget::initializeGL()
{
    initializeOpenGLFunctions();
    glClearColor(255, 0, 0, 0);
}

void QYUVOpenGLWidget::resizeGL(int w, int h)
{

}

void QYUVOpenGLWidget::paintGL()
{

}
```

在实现的虚函数`initializeGL()`方法中首先要执行`initializeOpenGLFunctions()`方法，来进行初始化

并且可以通过`glClearColor()`方法来设置窗口的背景色

在实例化这个`QYUVOpenGLWidget`时，在构造方法中需要传递一个空指针`Q_NULLPTR`，这样程序就会新建一个窗口来进行openGL绘制

## openGL学习

可以搜索`learnOpenGL cn`来进行学习

## yuv渲染

### 头文件介绍

```c++
#ifndef QYUVOPENGLWIDGET_H
#define QYUVOPENGLWIDGET_H
#include <QOpenGLWidget>
#include <QOpenGLFunctions>
#include <QOpenGLShaderProgram>
#include <QOpenGLBuffer>

class QYUVOpenGLWidget : public QOpenGLWidget, protected QOpenGLFunctions
{
    Q_OBJECT
public:
    explicit QYUVOpenGLWidget(QWidget *parent = nullptr);
    virtual ~QYUVOpenGLWidget();

    // QWidget虚函数
    QSize minimumSizeHint() const override;
    QSize sizeHint() const override;

    // 设置视频帧尺寸，用于生成纹理
    void setFrameSize(const QSize& frameSize);
    // 返回当前视频帧尺寸
    const QSize& frameSize();
    // 渲染视频帧
    void updateTextures(quint8* dataY, quint8* dataU, quint8* dataV, quint32 linesizeY, quint32 linesizeU, quint32 linesizeV);

protected:
    void initializeGL();
    void paintGL();
    void resizeGL(int width, int height);

private:    
    // 初始化着色器
    void initShader();
    // 初始化纹理
    void initTextures();
    // 清理纹理
    void deInitTextures();
    // 更新纹理
    void updateTexture(GLuint texture, quint32 textureType, quint8* pixels, quint32 stride);

private:
    // 视频帧尺寸
    QSize m_frameSize = {-1, -1};
    // 是否更新标志
    bool m_needUpdate = false;
    // 纹理初始化标志
    bool m_textureInited = false;

    // 顶点缓冲对象(Vertex Buffer Objects, VBO)：默认即为VertexBuffer(GL_ARRAY_BUFFER)类型
    QOpenGLBuffer m_vbo;

    // 着色器程序：编译链接着色器
    QOpenGLShaderProgram m_shaderProgram;    

    // YUV纹理，用于生成纹理贴图
    GLuint m_texture[3] = {0};
};

#endif // QYUVOPENGLWIDGET_H
```

