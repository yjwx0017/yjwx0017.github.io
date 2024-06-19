title: OSG 笔记
categories: 日志
tags: [OSG]
date: 2022-02-27 09:47:00
---
《OpenSceneGraph 3.0 Beginners Guide》 笔记
OSG-3.6.5 + VS2017

## Hello World

``` c++
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

int main()
{
    osg::ref_ptr<osg::Node> root = osgDB::readNodeFile("cessna.osg");

    osgViewer::Viewer viewer;
    viewer.setSceneData(root.get());

    return viewer.run();
}
```

<!--more-->

## osg::ArgumentParser

``` c++
osg::ArgumentParser arguments(&argc, argv);
std::string filename;
arguments.read("--model", filename);
```

## osg::notify()

``` c++
osg::notify(osg::WARN) << "Some warn message." << std::endl;
```

默认输出到std::cout和std::cerr，也可使用以下方法输出到日志文件：

``` c++
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>
#include <fstream>

class LogFileHandler : public osg::NotifyHandler
{
public:
    LogFileHandler(const std::string &file)
    {
        _log.open(file.c_str());
    }

    virtual ~LogFileHandler()
    {
        _log.close();
    }

    virtual void notify(osg::NotifySeverity severity, const char *msg)
    {
        _log << msg;
    }

protected:
    std::ofstream _log;
};

int main(int argc, char **argv)
{
    osg::setNotifyLevel(osg::INFO);
    osg::setNotifyHandler(new LogFileHandler("output.txt"));

    osg::ArgumentParser arguments(&argc, argv);
    osg::ref_ptr<osg::Node> root = osgDB::readNodeFiles(arguments);
    if (!root) {
        OSG_FATAL << arguments.getApplicationName() << ": No data loaded." << std::endl;
        return -1;
    }
    
    osgViewer::Viewer viewer;
    viewer.setSceneData(root.get());
    return viewer.run();
}
```

## 内置的基本几何体

``` c++
#include <osg/ShapeDrawable>
#include <osg/Geode>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::ShapeDrawable> shape1 = new osg::ShapeDrawable;
    shape1->setShape(new osg::Box(osg::Vec3(-3.0f, 0.0f, 0.0f), 2.0f, 2.0f, 1.0f));

    osg::ref_ptr<osg::ShapeDrawable> shape2 = new osg::ShapeDrawable;
    shape2->setShape(new osg::Sphere(osg::Vec3(3.0f, 0.0f, 0.0f), 1.0f));
    shape2->setColor(osg::Vec4(0.0f, 0.0f, 1.0f, 1.0f));

    osg::ref_ptr<osg::ShapeDrawable> shape3 = new osg::ShapeDrawable;
    shape3->setShape(new osg::Cone(osg::Vec3(0.0f, 0.0f, 0.0f), 1.0f, 1.0f));
    shape3->setColor(osg::Vec4(0.0f, 1.0f, 0.0f, 1.0f));

    osg::ref_ptr<osg::Geode> root = new osg::Geode;
    root->addDrawable(shape1);
    root->addDrawable(shape2);
    root->addDrawable(shape3);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

osg::ShapeDrawable 类并不是绘制几何体最有效率的方式，如果需要高性能，osg::Geometry 类是更好的选择。

## osg::Geometry

``` c++
#include <osg/Geometry>
#include <osg/Geode>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    // 四边形的四个顶点
    osg::ref_ptr<osg::Vec3Array> vertices = new osg::Vec3Array;
    vertices->push_back(osg::Vec3(0.0f, 0.0f, 0.0f));
    vertices->push_back(osg::Vec3(1.0f, 0.0f, 0.0f));
    vertices->push_back(osg::Vec3(1.0f, 0.0f, 1.0f));
    vertices->push_back(osg::Vec3(0.0f, 0.0f, 1.0f));

    // 法线
    osg::ref_ptr<osg::Vec3Array> normals = new osg::Vec3Array;
    normals->push_back(osg::Vec3(0.0f, -1.0f, 0.0f));

    // 每个顶点的颜色
    osg::ref_ptr<osg::Vec4Array> colors = new osg::Vec4Array;
    colors->push_back(osg::Vec4(1.0f, 0.0f, 0.0f, 1.0f));
    colors->push_back(osg::Vec4(0.0f, 1.0f, 0.0f, 1.0f));
    colors->push_back(osg::Vec4(0.0f, 0.0f, 1.0f, 1.0f));
    colors->push_back(osg::Vec4(1.0f, 1.0f, 1.0f, 1.0f));

    // 将数据设置到Geometry
    osg::ref_ptr<osg::Geometry> quad = new osg::Geometry;
    quad->setVertexArray(vertices);
    quad->setNormalArray(normals);                         // 设置法线数据
    quad->setNormalBinding(osg::Geometry::BIND_OVERALL);   // 表示只有一个法线向量
    quad->setColorArray(colors);                           // 设置颜色数据
    quad->setColorBinding(osg::Geometry::BIND_PER_VERTEX); // 表示每个顶点有一个颜色向量

    // 表示以上数据用于绘制四边形
    quad->addPrimitiveSet(new osg::DrawArrays(GL_QUADS, 0, 4));

    // 将Geometry添加到场景图结点
    osg::ref_ptr<osg::Geode> root = new osg::Geode;
    root->addDrawable(quad);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 使用顶点索引

``` c++
#include <osg/Geometry>
#include <osg/Geode>
#include <osgUtil/SmoothingVisitor>
#include <osgViewer/Viewer>

// 绘制一个八面体
int main(int argc, char **argv)
{
    // 顶点数组
    osg::ref_ptr<osg::Vec3Array> vertices = new osg::Vec3Array(6);
    (*vertices)[0].set( 0.0f,  0.0f,  1.0f);
    (*vertices)[1].set(-0.5f, -0.5f,  0.0f);
    (*vertices)[2].set( 0.5f, -0.5f,  0.0f);
    (*vertices)[3].set( 0.5f,  0.5f,  0.0f);
    (*vertices)[4].set(-0.5f,  0.5f,  0.0f);
    (*vertices)[5].set( 0.0f,  0.0f, -1.0f);

    // 8个三角形，每个三角形3个顶点索引，一共24个索引
    osg::ref_ptr<osg::DrawElementsUInt> indices = new osg::DrawElementsUInt(GL_TRIANGLES, 24);
    (*indices)[0]  = 0; (*indices)[1]  = 1; (*indices)[2]  = 2;
    (*indices)[3]  = 0; (*indices)[4]  = 2; (*indices)[5]  = 3;
    (*indices)[6]  = 0; (*indices)[7]  = 3; (*indices)[8]  = 4;
    (*indices)[9]  = 0; (*indices)[10] = 4; (*indices)[11] = 1;
    (*indices)[12] = 5; (*indices)[13] = 2; (*indices)[14] = 1;
    (*indices)[15] = 5; (*indices)[16] = 3; (*indices)[17] = 2;
    (*indices)[18] = 5; (*indices)[19] = 4; (*indices)[20] = 3;
    (*indices)[21] = 5; (*indices)[22] = 1; (*indices)[23] = 4;

    osg::ref_ptr<osg::Geometry> geom = new osg::Geometry;
    geom->setVertexArray(vertices);  
    geom->addPrimitiveSet(indices);

    // 自动生成法线
    osgUtil::SmoothingVisitor::smooth(*geom); 

    osg::ref_ptr<osg::Geode> root = new osg::Geode;
    root->addDrawable(geom);
    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## osgUtil::Tessellator

OpenGL 无法正确绘制凹多边形、自相交多边形和有孔多边形，osgUtil::Tessellator 可以将多边形处理为合法的多边形。

``` c++
#include <osg/Geometry>
#include <osg/Geode>
#include <osgUtil/Tessellator>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Vec3Array> vertices = new osg::Vec3Array;
    vertices->push_back(osg::Vec3(0.0f, 0.0f, 0.0f));
    vertices->push_back(osg::Vec3(2.0f, 0.0f, 0.0f));
    vertices->push_back(osg::Vec3(2.0f, 0.0f, 1.0f));
    vertices->push_back(osg::Vec3(1.0f, 0.0f, 1.0f));
    vertices->push_back(osg::Vec3(1.0f, 0.0f, 2.0f));
    vertices->push_back(osg::Vec3(2.0f, 0.0f, 2.0f));
    vertices->push_back(osg::Vec3(2.0f, 0.0f, 3.0f));
    vertices->push_back(osg::Vec3(0.0f, 0.0f, 3.0f));

    osg::ref_ptr<osg::Vec3Array> normals = new osg::Vec3Array;
    normals->push_back(osg::Vec3(0.0f, -1.0f, 0.0f));

    osg::ref_ptr<osg::Geometry> geom = new osg::Geometry;
    geom->setVertexArray(vertices.get());
    geom->setNormalArray(normals.get());
    geom->setNormalBinding(osg::Geometry::BIND_OVERALL);
    geom->addPrimitiveSet(new osg::DrawArrays(GL_POLYGON, 0, 8));

    osgUtil::Tessellator tessellator;
    tessellator.retessellatePolygons(*geom);

    osg::ref_ptr<osg::Geode> root = new osg::Geode;
    root->addDrawable(geom.get());
    osgViewer::Viewer viewer;
    viewer.setSceneData(root.get());
    return viewer.run();
}
```

## osg::TriangleFunctor

使用 osg::TriangleFunctor 访问 Geometry 的三角形信息。

``` c++
#include <osg/Geometry>
#include <osg/Geode>
#include <osgViewer/Viewer>
#include <osg/TriangleFunctor>
#include <iostream>

std::ostream& operator<<(std::ostream &stream, const osg::Vec3 &vec)
{
    stream << vec[0] << " " << vec[1] << " " << vec[2];
    return stream;
}

struct FaceCollector
{
    void operator()(const osg::Vec3 &v1, const osg::Vec3 &v2, const osg::Vec3 &v3)
    {
        std::cout << "Face vertices: " << v1 << "; " << v2 << "; " << v3 << std::endl;
    }
};

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Vec3Array> vertices = new osg::Vec3Array;
    vertices->push_back(osg::Vec3(0.0f, 0.0f, 0.0f));
    vertices->push_back(osg::Vec3(0.0f, 0.0f, 1.0f));
    vertices->push_back(osg::Vec3(1.0f, 0.0f, 0.0f));
    vertices->push_back(osg::Vec3(1.0f, 0.0f, 1.5f));
    vertices->push_back(osg::Vec3(2.0f, 0.0f, 0.0f));
    vertices->push_back(osg::Vec3(2.0f, 0.0f, 1.0f));
    vertices->push_back(osg::Vec3(3.0f, 0.0f, 0.0f));
    vertices->push_back(osg::Vec3(3.0f, 0.0f, 1.5f));
    vertices->push_back(osg::Vec3(4.0f, 0.0f, 0.0f));
    vertices->push_back(osg::Vec3(4.0f, 0.0f, 1.0f));

    osg::ref_ptr<osg::Vec3Array> normals = new osg::Vec3Array;
    normals->push_back(osg::Vec3(0.0f, -1.0f, 0.0f));

    osg::ref_ptr<osg::Geometry> geom = new osg::Geometry;
    geom->setVertexArray(vertices.get());
    geom->setNormalArray(normals.get());
    geom->setNormalBinding(osg::Geometry::BIND_OVERALL);
    geom->addPrimitiveSet(new osg::DrawArrays(GL_QUAD_STRIP, 0, 10));

    osg::TriangleFunctor<FaceCollector> functor;
    geom->accept(functor);

    osg::ref_ptr<osg::Geode> root = new osg::Geode;
    root->addDrawable(geom.get());

    osgViewer::Viewer viewer;
    viewer.setSceneData(root.get());
    return viewer.run();
}
```

## 继承 osg::Drawable

可在 drawImplementation() 函数中使用 OpenGL 进行绘制。

运行错误：调试显示 CurrentWindow 为空（使用freeglut）。

``` c++
#include <GL/glut.h>
#include <osg/Drawable>
#include <osg/Geode>
#include <osgViewer/Viewer>

class TeapotDrawable : public osg::Drawable
{
public:
    TeapotDrawable(float size = 1.0f) : m_size(size) {}
    TeapotDrawable(const TeapotDrawable &copy, const osg::CopyOp &copyop = osg::CopyOp::SHALLOW_COPY)
        : osg::Drawable(copy, copyop), m_size(copy.m_size)
    {
    }

    META_Object(osg, TeapotDrawable);

    virtual osg::BoundingBox computeBoundingBox() const override
    {
        osg::Vec3 min(-m_size, -m_size, -m_size);
        osg::Vec3 max(m_size, m_size, m_size);
        return osg::BoundingBox(min, max);
    }

    virtual void drawImplementation(osg::RenderInfo &) const override
    {
        glFrontFace(GL_CW);
        glutSolidTeapot(m_size);
        glFrontFace(GL_CCW);
    }

private:
    float m_size;
};

int main(int argc, char **argv)
{
    glutInit(&argc, argv);

    osg::ref_ptr<osg::Geode> root = new osg::Geode;
    root->addDrawable(new TeapotDrawable(1.0f));

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 添加多个模型到场景图

``` c++
#include <osg/Group>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    // 加载文件
    osg::ref_ptr<osg::Node> model1 = osgDB::readNodeFile("cessna.osg");
    osg::ref_ptr<osg::Node> model2 = osgDB::readNodeFile("cow.osg");

    // 将模型添加到根结点
    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(model1);
    root->addChild(model2);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## osg::Node 向下转型

``` c++
// Assumes the Cessna's root node is a group node.
osg::ref_ptr<osg::Node> model = osgDB::readNodeFile("cessna.osg");
osg::Group* convModel1 = model->asGroup(); // OK!
osg::Geode* convModel2 = model->asGeode(); // Returns NULL.
```

使用 dynamic_cast<> 也可以。

## 对子结点应用变换

``` c++
// 
#include <osg/MatrixTransform>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    // 加载模型
    osg::ref_ptr<osg::Node> model = osgDB::readNodeFile("cessna.osg");

    // 模型实例1
    osg::ref_ptr<osg::MatrixTransform> transform1 = new osg::MatrixTransform;
    transform1->setMatrix(osg::Matrix::translate(-25.0, 0.0, 0.0));
    transform1->addChild(model);

    // 模型实例2
    osg::ref_ptr<osg::MatrixTransform> transform2 = new osg::MatrixTransform;
    transform2->setMatrix(osg::Matrix::translate(25.0, 0.0, 0.0));
    transform2->addChild(model);

    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(transform1);
    root->addChild(transform2);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

osg::PositionAttitudeTransform 比 osg::MatrixTransform 更易于使用，setPosition()、setRotate() 等函数可以更加方便地设置变换。

## 使用 osg::Switch 切换结点的显示和隐藏

``` c++
#include <osg/Switch>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Node> model1 = osgDB::readNodeFile("cessna.osg");
    osg::ref_ptr<osg::Node> model2 = osgDB::readNodeFile("cessnafire.osg");

    osg::ref_ptr<osg::Switch> root = new osg::Switch;
    root->addChild(model1, false);
    root->addChild(model2, true);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 使用 osg::LOD

``` c++
#include <osg/LOD>
#include <osgDB/ReadFile>
#include <osgUtil/Simplifier>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Node> modelL3 = osgDB::readNodeFile("cessna.osg");
    osg::ref_ptr<osg::Node> modelL2 = modelL3->clone(osg::CopyOp::DEEP_COPY_ALL)->asNode();
    osg::ref_ptr<osg::Node> modelL1 = modelL3->clone(osg::CopyOp::DEEP_COPY_ALL)->asNode();

    // 降低模型分辨率
    osgUtil::Simplifier simplifier;
    simplifier.setSampleRatio(0.5);
    modelL2->accept(simplifier);

    // SampleRatio值越小模型分辨率越低
    simplifier.setSampleRatio(0.1);
    modelL1->accept(simplifier);

    osg::ref_ptr<osg::LOD> root = new osg::LOD;
    root->addChild(modelL1, 200.0f, FLT_MAX);
    root->addChild(modelL2, 50.0f, 200.0f);
    root->addChild(modelL3, 0.0f, 50.0f);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## osg::ProxyNode

osg::ProxyNode 结点设置的文件不会立即加载，而是在 Viewer 准备好后，在需要时动态加载。

``` c++
#include <osg/ProxyNode>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::ProxyNode> model1 = new osg::ProxyNode;
    model1->setFileName(0, "cow.osg");

    osg::ref_ptr<osg::Node> model2 = osgDB::readNodeFile("cessna.osg");

    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(model1);
    root->addChild(model2);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

osg::PagedLOD 可以设置动态加载的 LOD 模型：

``` c++
osg::ref_ptr<osg::PagedLOD> pagedLOD = new osg::PagedLOD;
pagedLOD->addChild( modelL1, 200.0f, FLT_MAX ); 
pagedLOD->setFileName( 1, "cessna.osg" );  // 动态加载的模型
pagedLOD->setRange( 1, 0.0f, 200.0f );
```

## 重写 traverse() 函数

traverse() 函数每帧被调用，以下代码实现每60帧切换模型：

``` c++
#include <osg/Switch>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

class AnimatingSwitch : public osg::Switch 
{
public:
    AnimatingSwitch() : m_count(0) {}
    AnimatingSwitch(const AnimatingSwitch &copy, const osg::CopyOp &copyop = osg::CopyOp::SHALLOW_COPY)
        : osg::Switch(copy, copyop), m_count(copy.m_count)
    {}

    META_Node(osg, AnimatingSwitch);

    virtual void traverse(osg::NodeVisitor &nv) override
    {
        if (!(++m_count % 60)) {
            setValue(0, !getValue(0));
            setValue(1, !getValue(1));
        }

        osg::Switch::traverse(nv);
    }

protected:
    uint32_t m_count;
};

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Node> model1 = osgDB::readNodeFile("cessna.osg");
    osg::ref_ptr<osg::Node> model2 = osgDB::readNodeFile("cessnafire.osg");

    osg::ref_ptr<AnimatingSwitch> root = new AnimatingSwitch;
    root->addChild(model1, true);
    root->addChild(model2, false);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 本地坐标转为世界坐标

``` c++
osg::Vec3 posInWorld = node->getBound().center() * 
    osg::computeLocalToWorld(node->getParentalNodePaths()[0]);
```

## Visitor

``` c++
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>
#include <iostream>

class InfoVisitor : public osg::NodeVisitor
{
public:
    InfoVisitor() : m_level(0)
    {
        setTraversalMode(osg::NodeVisitor::TRAVERSE_ALL_CHILDREN);
    }

    std::string spaces()
    {
        return std::string(m_level * 2, ' ');
    }

    virtual void apply(osg::Node &node) override
    {
        std::cout << spaces() << node.libraryName() << "::"
            << node.className() << std::endl;
        m_level++;
        traverse(node);
        m_level--;
    }

    virtual void apply(osg::Geode &geode) override
    {
        std::cout << spaces() << geode.libraryName() << "::"
            << geode.className() << std::endl;
        m_level++;
        for (unsigned int i = 0; i < geode.getNumDrawables(); ++i) {
            osg::Drawable* drawable = geode.getDrawable(i);
            std::cout << spaces() << drawable->libraryName() << "::"
                << drawable->className() << std::endl;
        }
        traverse(geode);
        m_level--;
    }

protected:
    unsigned int m_level;
};

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Node> root = osgDB::readNodeFile("cessnafire.osg");

    InfoVisitor infoVisitor;
    root->accept(infoVisitor);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 加载命令行中指定的模型文件

``` c++
osg::ArgumentParser arguments(&argc, argv);
osg::ref_ptr<osg::Node> root = osgDB::readNodeFiles(arguments);
if (!root) {
    OSG_FATAL << arguments.getApplicationName() << ": No data loaded." << std::endl;
    return -1;
}
```

## 设置 StateAttribute

``` c++
#include <osg/PolygonMode>
#include <osg/MatrixTransform>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Node> model = osgDB::readNodeFile("cessna.osg");

    osg::ref_ptr<osg::MatrixTransform> transformation1 = new osg::MatrixTransform;
    transformation1->setMatrix(osg::Matrix::translate(-25.0f, 0.0f, 0.0f));
    transformation1->addChild(model.get());

    osg::ref_ptr<osg::MatrixTransform> transformation2 = new osg::MatrixTransform;
    transformation2->setMatrix(osg::Matrix::translate(25.0f, 0.0f, 0.0f));
    transformation2->addChild(model.get());

    // 设置多边形模式，双面绘制、线框模式
    osg::ref_ptr<osg::PolygonMode> pm = new osg::PolygonMode;
    pm->setMode(osg::PolygonMode::FRONT_AND_BACK, osg::PolygonMode::LINE);
    transformation1->getOrCreateStateSet()->setAttribute(pm);

    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(transformation1.get());
    root->addChild(transformation2.get());

    osgViewer::Viewer viewer;
    viewer.setSceneData(root.get());
    return viewer.run();
}
```

## osg::StateAttribute::OVERRIDE 和 osg::StateAttribute::PROTECTED

OVERRIDE 强制所有子结点继承其属性，如果子结点想要打破这个限制需要使用 PROTECTED 标志。

``` c++
#include <osg/PolygonMode>
#include <osg/MatrixTransform>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Node> model = osgDB::readNodeFile("glider.osg");

    osg::ref_ptr<osg::MatrixTransform> transformation1 = new osg::MatrixTransform;
    transformation1->setMatrix(osg::Matrix::translate(-0.5f, 0.0f, 0.0f));
    transformation1->addChild(model);

    osg::ref_ptr<osg::MatrixTransform> transformation2 = new osg::MatrixTransform;
    transformation2->setMatrix(osg::Matrix::translate(0.5f, 0.0f, 0.0f));
    transformation2->addChild(model);

    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(transformation1);
    root->addChild(transformation2);

    // 根结点强制启用光照
    root->getOrCreateStateSet()->setMode(GL_LIGHTING,
        osg::StateAttribute::ON | osg::StateAttribute::OVERRIDE);

    // 子结点关闭光照（无效）
    transformation1->getOrCreateStateSet()->setMode(GL_LIGHTING, osg::StateAttribute::OFF);

    // 子结点关闭光照（有效，使用了PROTECTED）
    transformation2->getOrCreateStateSet()->setMode(GL_LIGHTING, 
        osg::StateAttribute::OFF | osg::StateAttribute::PROTECTED);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## osg::Fog 雾效果

``` c++
#include <osg/Fog>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Fog> fog = new osg::Fog;
    fog->setMode(osg::Fog::LINEAR);
    fog->setStart(500.0f);
    fog->setEnd(2500.0f);
    fog->setColor(osg::Vec4(1.0f, 1.0f, 0.0f, 1.0f));

    osg::ref_ptr<osg::Node> model = osgDB::readNodeFile("lz.osg");
    model->getOrCreateStateSet()->setAttributeAndModes(fog);

    osgViewer::Viewer viewer;
    viewer.setSceneData(model);
    return viewer.run();
}
```

## osg::Texture2D 纹理

OpenGL纹理坐标系左下角为原点。

``` c++
#include <osg/Texture2D>
#include <osg/Geometry>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    // 顶点数组
    osg::ref_ptr<osg::Vec3Array> vertices = new osg::Vec3Array;
    vertices->push_back(osg::Vec3(-0.5f, 0.0f, -0.5f));
    vertices->push_back(osg::Vec3( 0.5f, 0.0f, -0.5f));
    vertices->push_back(osg::Vec3( 0.5f, 0.0f,  0.5f));
    vertices->push_back(osg::Vec3(-0.5f, 0.0f,  0.5f));

    // 法线
    osg::ref_ptr<osg::Vec3Array> normals = new osg::Vec3Array;
    normals->push_back(osg::Vec3(0.0f, -1.0f, 0.0f));

    // 纹理坐标
    osg::ref_ptr<osg::Vec2Array> texcoords = new osg::Vec2Array;
    texcoords->push_back(osg::Vec2(0.0f, 0.0f));
    texcoords->push_back(osg::Vec2(1.0f, 0.0f));
    texcoords->push_back(osg::Vec2(1.0f, 1.0f));
    texcoords->push_back(osg::Vec2(0.0f, 1.0f));

    osg::ref_ptr<osg::Geometry> quad = new osg::Geometry;
    quad->setVertexArray(vertices);
    quad->setNormalArray(normals);
    quad->setNormalBinding(osg::Geometry::BIND_OVERALL);
    quad->setTexCoordArray(0, texcoords);
    quad->addPrimitiveSet(new osg::DrawArrays(GL_QUADS, 0, 4));

    // 创建纹理对象
    osg::ref_ptr<osg::Texture2D> texture = new osg::Texture2D;
    osg::ref_ptr<osg::Image> image = osgDB::readImageFile("Images/lz.rgb");
    texture->setImage(image);

    osg::ref_ptr<osg::Geode> root = new osg::Geode;
    root->addDrawable(quad);

    // 启用纹理
    root->getOrCreateStateSet()->setTextureAttributeAndModes(0, texture);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 实现半透明

``` c++
#include <osg/BlendFunc>
#include <osg/Texture2D>
#include <osg/Geometry>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    // 顶点数组
    osg::ref_ptr<osg::Vec3Array> vertices = new osg::Vec3Array;
    vertices->push_back(osg::Vec3(-0.5f, 0.0f, -0.5f));
    vertices->push_back(osg::Vec3( 0.5f, 0.0f, -0.5f));
    vertices->push_back(osg::Vec3( 0.5f, 0.0f,  0.5f));
    vertices->push_back(osg::Vec3(-0.5f, 0.0f,  0.5f));

    // 法线
    osg::ref_ptr<osg::Vec3Array> normals = new osg::Vec3Array;
    normals->push_back(osg::Vec3(0.0f, -1.0f, 0.0f));

    // 纹理坐标
    osg::ref_ptr<osg::Vec2Array> texcoords = new osg::Vec2Array;
    texcoords->push_back(osg::Vec2(0.0f, 0.0f));
    texcoords->push_back(osg::Vec2(1.0f, 0.0f));
    texcoords->push_back(osg::Vec2(1.0f, 1.0f));
    texcoords->push_back(osg::Vec2(0.0f, 1.0f));

    osg::ref_ptr<osg::Vec4Array> colors = new osg::Vec4Array;
    colors->push_back(osg::Vec4(1.0f, 1.0f, 1.0f, 0.5f));

    osg::ref_ptr<osg::Geometry> quad = new osg::Geometry;
    quad->setVertexArray(vertices);
    quad->setNormalArray(normals);
    quad->setNormalBinding(osg::Geometry::BIND_OVERALL);
    quad->setColorArray(colors);
    quad->setColorBinding(osg::Geometry::BIND_OVERALL);
    quad->setTexCoordArray(0, texcoords);
    quad->addPrimitiveSet(new osg::DrawArrays(GL_QUADS, 0, 4));

    osg::ref_ptr<osg::Geode> geode = new osg::Geode;
    geode->addDrawable(quad);

    // 纹理对象
    osg::ref_ptr<osg::Texture2D> texture = new osg::Texture2D;
    osg::ref_ptr<osg::Image> image = osgDB::readImageFile("Images/lz.rgb");
    texture->setImage(image);

    osg::ref_ptr<osg::BlendFunc> blendFunc = new osg::BlendFunc;
    blendFunc->setFunction(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);

    osg::StateSet* stateset = geode->getOrCreateStateSet();
    stateset->setTextureAttributeAndModes(0, texture);  // 启用纹理
    stateset->setAttributeAndModes(blendFunc);          // 设置混合模式

    // 使OSG调整渲染顺序以便最后绘制半透明的物体
    stateset->setRenderingHint(osg::StateSet::TRANSPARENT_BIN);

    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(geode);
    root->addChild(osgDB::readNodeFile("glider.osg"));

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 着色器：卡通效果

``` c++
#include <osg/Program>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

// 顶点着色器
static const char* vertSource = {
    "varying vec3 normal;\n"
    "void main()\n"
    "{\n"
    "    normal = normalize(gl_NormalMatrix * gl_Normal);\n"
    "    gl_Position = ftransform();\n"
    "}\n"
};

// 片段着色器
// 根据法线和光线的夹角从四种颜色中选择一种
static const char* fragSource = {
    "uniform vec4 color1;\n"
    "uniform vec4 color2;\n"
    "uniform vec4 color3;\n"
    "uniform vec4 color4;\n"
    "varying vec3 normal;\n"
    "void main()\n"
    "{\n"
    "    float intensity = dot(vec3(gl_LightSource[0].position), normal); \n"
    "    if (intensity > 0.95) gl_FragColor = color1;\n"
    "    else if (intensity > 0.5) gl_FragColor = color2;\n"
    "    else if (intensity > 0.25) gl_FragColor = color3;\n"
    "    else gl_FragColor = color4;\n"
    "}\n"
};

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Shader> vertShader = new osg::Shader(osg::Shader::VERTEX, vertSource);
    osg::ref_ptr<osg::Shader> fragShader = new osg::Shader(osg::Shader::FRAGMENT, fragSource);
    osg::ref_ptr<osg::Program> program = new osg::Program;
    program->addShader(vertShader.get());
    program->addShader(fragShader.get());

    osg::ref_ptr<osg::Node> model = osgDB::readNodeFile("cow.osg");

    osg::StateSet* stateset = model->getOrCreateStateSet();
    stateset->setAttributeAndModes(program.get());  // 启用着色器

    // 设置着色器变量
    stateset->addUniform(new osg::Uniform("color1", osg::Vec4(1.0f, 0.5f, 0.5f, 1.0f)));
    stateset->addUniform(new osg::Uniform("color2", osg::Vec4(0.5f, 0.2f, 0.2f, 1.0f)));
    stateset->addUniform(new osg::Uniform("color3", osg::Vec4(0.2f, 0.1f, 0.1f, 1.0f)));
    stateset->addUniform(new osg::Uniform("color4", osg::Vec4(0.1f, 0.05f, 0.05f, 1.0f)));

    osgViewer::Viewer viewer;
    viewer.setSceneData(model);
    return viewer.run();
}
```

## 实现 HUD

``` c++
#include <osg/Camera>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Node> model = osgDB::readNodeFile("lz.osg");
    osg::ref_ptr<osg::Node> hudModel = osgDB::readNodeFile("glider.osg");

    osg::ref_ptr<osg::Camera> camera = new osg::Camera;
    camera->setClearMask(GL_DEPTH_BUFFER_BIT);         // 只清空深度缓冲区
    camera->setRenderOrder(osg::Camera::POST_RENDER);  // 在主Camera之后渲染

    camera->setReferenceFrame(osg::Camera::ABSOLUTE_RF);
    camera->setViewMatrixAsLookAt(
        osg::Vec3(0.0f, -5.0f, 5.0f), osg::Vec3(), osg::Vec3(0.0f, 1.0f, 1.0f));
    camera->addChild(hudModel);

    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(model);
    root->addChild(camera);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 自定义渲染循环

``` c++
#include <osgDB/ReadFile>
#include <osgGA/TrackballManipulator>
#include <osgViewer/Viewer>
#include <iostream>

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Node> model = osgDB::readNodeFile("lz.osg");

    osgViewer::Viewer viewer;
    viewer.setSceneData(model);
    viewer.setCameraManipulator(new osgGA::TrackballManipulator);

    while (!viewer.done()) {
        viewer.frame();
        std::cout << "Frame number: " << viewer.getFrameStamp()->getFrameNumber() << std::endl;
    }

    return 0;
}
```

## 窗口模式

``` c++
viewer.setUpViewInWindow(50, 50, 800, 600);
```

## CompositeViewer

``` c++
#include <osgDB/ReadFile>
#include <osgViewer/CompositeViewer>

osgViewer::View* createView(int x, int y, int w, int h, osg::Node *scene)
{
    osg::ref_ptr<osgViewer::View> view = new osgViewer::View;
    view->setUpViewInWindow(x, y, w, h);
    view->setSceneData(scene);
    return view.release();
}

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Node> model1 = osgDB::readNodeFile("cessna.osg");
    osg::ref_ptr<osg::Node> model2 = osgDB::readNodeFile("cow.osg");
    osg::ref_ptr<osg::Node> model3 = osgDB::readNodeFile("glider.osg");

    osgViewer::CompositeViewer viewer;
    viewer.addView(createView(50, 50, 320, 240, model1));
    viewer.addView(createView(370, 50, 320, 240, model2));
    viewer.addView(createView(185, 310, 320, 240, model3));
    return viewer.run();
}
```

## 启用多重采样抗锯齿

``` c++
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    osg::DisplaySettings::instance()->setNumMultiSamples(4);

    osg::ref_ptr<osg::Node> model = osgDB::readNodeFile("cessna.osg");

    osgViewer::Viewer viewer;
    viewer.setSceneData(model.get());
    return viewer.run();
}
```

## 实现红蓝3D

``` c++
osg::DisplaySettings::instance()->setStereoMode(osg::DisplaySettings::ANAGLYPHIC);
osg::DisplaySettings::instance()->setEyeSeparation(0.05f);
osg::DisplaySettings::instance()->setStereo(true);
```

## 渲染到纹理

``` c++
#include <osg/Camera>
#include <osg/Texture2D>
#include <osgDB/ReadFile>
#include <osgGA/TrackballManipulator>
#include <osgViewer/Viewer>

class FindTextureVisitor : public osg::NodeVisitor
{
public:
    FindTextureVisitor(osg::Texture *tex) : m_texture(tex)
    {
        setTraversalMode(osg::NodeVisitor::TRAVERSE_ALL_CHILDREN);
    }

    virtual void apply(osg::Node &node) override
    {
        replaceTexture(node.getStateSet());
        traverse(node);
    }

    virtual void apply(osg::Geode &geode) override
    {
        replaceTexture(geode.getStateSet());
        for (unsigned int i = 0; i < geode.getNumDrawables(); ++i) {
            replaceTexture(geode.getDrawable(i)->getStateSet());
        }
        traverse(geode);
    }

    void replaceTexture(osg::StateSet *ss)
    {
        if (ss) {
            osg::Texture *oldTexture = dynamic_cast<osg::Texture*>(
                ss->getTextureAttribute(0, osg::StateAttribute::TEXTURE));

            if (oldTexture) 
                ss->setTextureAttribute(0, m_texture);
        }
    }

protected:
    osg::ref_ptr<osg::Texture> m_texture;
};

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Node> model = osgDB::readNodeFile("lz.osg");
    osg::ref_ptr<osg::Node> subModel = osgDB::readNodeFile("glider.osg");

    int texWidth = 1024;
    int texHeight = 1024;

    // 创建纹理对象
    osg::ref_ptr<osg::Texture2D> texture = new osg::Texture2D;
    texture->setTextureSize(texWidth, texHeight); 
    texture->setInternalFormat(GL_RGBA);
    texture->setFilter(osg::Texture2D::MIN_FILTER, osg::Texture2D::LINEAR);
    texture->setFilter(osg::Texture2D::MAG_FILTER, osg::Texture2D::LINEAR);

    // 将模型中的纹理替换为自定义纹理
    FindTextureVisitor visitor(texture);
    if (model.valid()) 
        model->accept(visitor);

    // 该摄像机将subModel渲染到纹理
    osg::ref_ptr<osg::Camera> camera = new osg::Camera;
    camera->setViewport(0, 0, texWidth, texHeight);
    camera->setClearColor(osg::Vec4(1.0f, 1.0f, 1.0f, 0.0f));
    camera->setClearMask(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    camera->setRenderOrder(osg::Camera::PRE_RENDER);
    camera->setRenderTargetImplementation(osg::Camera::FRAME_BUFFER_OBJECT);
    camera->attach(osg::Camera::COLOR_BUFFER, texture);
    camera->setReferenceFrame(osg::Camera::ABSOLUTE_RF);
    camera->addChild(subModel);

    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(model);
    root->addChild(camera);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    viewer.setCameraManipulator(new osgGA::TrackballManipulator);

    float delta = 0.1f, bias = 0.0f;
    osg::Vec3 eye(0.0f, -5.0f, 5.0f);

    while (!viewer.done())
    {
        if (bias < -1.0f) 
            delta = 0.1f;
        else if (bias > 1.0f) 
            delta = -0.1f;

        bias += delta;
        // 实现镜头摇晃效果
        // 非线程安全，应使用回调函数，后面将会介绍
        camera->setViewMatrixAsLookAt(eye, osg::Vec3(), osg::Vec3(bias, 1.0f, 1.0f));
        viewer.frame();
    }

    return 0;
}
```

保存到文件：

``` c++
osg::ref_ptr<osg::Image> image = new osg::Image;
image->allocateImage(width, height, 1, GL_RGBA, GL_UNSIGNED_BYTE);
camera->attach(osg::Camera::COLOR_BUFFER, image);
...
// After running for a while
osgDB::writeImageFile(*image, "saved_image.bmp");
```

## 使用回调函数

``` c++
#include <osg/Switch>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

class SwitchingCallback : public osg::NodeCallback
{
public:
    SwitchingCallback() : m_count(0) {}
    virtual void operator()(osg::Node *node, osg::NodeVisitor *nv) override
    {
        osg::Switch* switchNode = static_cast<osg::Switch*>(node);
        if (!(++m_count % 60) && switchNode)
        {
            switchNode->setValue(0, !switchNode->getValue(0));
            switchNode->setValue(1, !switchNode->getValue(1));
        }
        traverse(node, nv);
    }

protected:
    unsigned int m_count;
};

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Node> model1 = osgDB::readNodeFile("cessna.osg");
    osg::ref_ptr<osg::Node> model2 = osgDB::readNodeFile("cessnafire.osg");

    osg::ref_ptr<osg::Switch> root = new osg::Switch;
    root->addChild(model1, false);
    root->addChild(model2, true);

    // 设置回调函数
    root->setUpdateCallback(new SwitchingCallback);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 动态更新模型数据

``` c++
#include <osg/Geometry>
#include <osg/Geode>
#include <osgViewer/Viewer>

osg::Geometry* createQuad()
{
    osg::ref_ptr<osg::Vec3Array> vertices = new osg::Vec3Array;
    vertices->push_back(osg::Vec3(0.0f, 0.0f, 0.0f));
    vertices->push_back(osg::Vec3(1.0f, 0.0f, 0.0f));
    vertices->push_back(osg::Vec3(1.0f, 0.0f, 1.0f));
    vertices->push_back(osg::Vec3(0.0f, 0.0f, 1.0f));

    osg::ref_ptr<osg::Vec3Array> normals = new osg::Vec3Array;
    normals->push_back(osg::Vec3(0.0f, -1.0f, 0.0f));

    osg::ref_ptr<osg::Vec4Array> colors = new osg::Vec4Array;
    colors->push_back(osg::Vec4(1.0f, 0.0f, 0.0f, 1.0f));
    colors->push_back(osg::Vec4(0.0f, 1.0f, 0.0f, 1.0f));
    colors->push_back(osg::Vec4(0.0f, 0.0f, 1.0f, 1.0f));
    colors->push_back(osg::Vec4(1.0f, 1.0f, 1.0f, 1.0f));

    osg::ref_ptr<osg::Geometry> quad = new osg::Geometry;
    quad->setVertexArray(vertices);
    quad->setNormalArray(normals);
    quad->setNormalBinding(osg::Geometry::BIND_OVERALL);
    quad->setColorArray(colors);
    quad->setColorBinding(osg::Geometry::BIND_PER_VERTEX);
    quad->addPrimitiveSet(new osg::DrawArrays(GL_QUADS, 0, 4));
    return quad.release();
}

class DynamicQuadCallback : public osg::Drawable::UpdateCallback
{
public:
    virtual void update(osg::NodeVisitor*, osg::Drawable *drawable) override
    {
        osg::Geometry *quad = static_cast<osg::Geometry*>(drawable);
        if (!quad) 
            return;

        osg::Vec3Array* vertices = static_cast<osg::Vec3Array*>(quad->getVertexArray());
        if (!vertices) 
            return;

        osg::Quat quat(osg::PI * 0.01, osg::X_AXIS);
        vertices->back() = quat * vertices->back();
        quad->dirtyDisplayList();
        quad->dirtyBound();
    }
};

int main(int argc, char **argv)
{
    osg::Geometry* quad = createQuad();
    quad->setDataVariance(osg::Object::DYNAMIC);  // 避免多线程冲突
    quad->setUpdateCallback(new DynamicQuadCallback);

    osg::ref_ptr<osg::Geode> root = new osg::Geode;
    root->addDrawable(quad);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root.get());
    return viewer.run();
}
```

## AnimationPath

``` c++
#include <osg/AnimationPath>
#include <osg/MatrixTransform>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

osg::AnimationPath* createAnimationPath(float radius, float time)
{
    osg::ref_ptr<osg::AnimationPath> path = new osg::AnimationPath;
    path->setLoopMode(osg::AnimationPath::LOOP);

    unsigned int numSamples = 32;
    float delta_yaw = 2.0f * osg::PI / ((float)numSamples - 1.0f);
    float delta_time = time / (float)numSamples;
    for (unsigned int i = 0; i < numSamples; ++i)
    {
        float yaw = delta_yaw * (float)i;
        osg::Vec3 pos(sinf(yaw)*radius, cosf(yaw)*radius, 0.0f);
        osg::Quat rot(-yaw, osg::Z_AXIS);
        path->insert(delta_time * (float)i, osg::AnimationPath::ControlPoint(pos, rot));
    }
    return path.release();
}

int main(int argc, char **argv)
{
    // 文件名后边的部分表示绕z轴旋转90度
    osg::ref_ptr<osg::Node> model = osgDB::readNodeFile("cessna.osg.0,0,90.rot");

    osg::ref_ptr<osg::MatrixTransform> root = new osg::MatrixTransform;
    root->addChild(model);

    osg::ref_ptr<osg::AnimationPathCallback> callback = new osg::AnimationPathCallback;
    callback->setAnimationPath(createAnimationPath(50.0f, 6.0f));
    root->setUpdateCallback(callback);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root.get());
    return viewer.run();
}
```

## 渐入渐出动画

``` c++
#include <osg/Geode>
#include <osg/Geometry>
#include <osg/BlendFunc>
#include <osg/Material>
#include <osgAnimation/EaseMotion>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

class AlphaFadingCallback : public osg::StateAttributeCallback
{
public:
    AlphaFadingCallback()
    {
        m_motion = new osgAnimation::InOutCubicMotion(0.0f, 1.0f);
    }

    virtual void operator()(osg::StateAttribute* sa, osg::NodeVisitor*) override
    {
        osg::Material* material = static_cast<osg::Material*>(sa);
        if (material)
        {
            m_motion->update(0.005);
            float alpha = m_motion->getValue();
            material->setDiffuse(osg::Material::FRONT_AND_BACK, osg::Vec4(0.0f, 1.0f, 1.0f, alpha));
        }
    }

protected:
    osg::ref_ptr<osgAnimation::InOutCubicMotion> m_motion;
};

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Drawable> quad = osg::createTexturedQuadGeometry(
        osg::Vec3(-0.5f, 0.0f, -0.5f), osg::Vec3(1.0f, 0.0f, 0.0f), osg::Vec3(0.0f, 0.0f, 1.0f));

    osg::ref_ptr<osg::Geode> geode = new osg::Geode;
    geode->addDrawable(quad);

    osg::ref_ptr<osg::Material> material = new osg::Material;
    material->setAmbient(osg::Material::FRONT_AND_BACK, osg::Vec4(0.0f, 0.0f, 0.0f, 1.0f));
    material->setDiffuse(osg::Material::FRONT_AND_BACK, osg::Vec4(0.0f, 1.0f, 1.0f, 0.5f));
    material->setUpdateCallback(new AlphaFadingCallback);

    geode->getOrCreateStateSet()->setAttributeAndModes(material);
    geode->getOrCreateStateSet()->setAttributeAndModes(new osg::BlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA));
    geode->getOrCreateStateSet()->setRenderingHint(osg::StateSet::TRANSPARENT_BIN);

    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(geode);
    root->addChild(osgDB::readNodeFile("glider.osg"));

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 纹理动画：使用图像序列

``` c++
#include <osg/ImageSequence>
#include <osg/Texture2D>
#include <osg/Geometry>
#include <osg/Geode>
#include <osgViewer/Viewer>

osg::Image* createSpotLight(const osg::Vec4& centerColor, 
    const osg::Vec4& bgColor, unsigned int size, float power)
{
    osg::ref_ptr<osg::Image> image = new osg::Image;
    image->allocateImage(size, size, 1, GL_RGBA, GL_UNSIGNED_BYTE);
    float mid = (float(size) - 1) * 0.5f;
    float div = 2.0f / float(size);
    for (unsigned int r = 0; r < size; ++r)
    {
        unsigned char* ptr = image->data(0, r);
        for (unsigned int c = 0; c < size; ++c)
        {
            float dx = (float(c) - mid)*div;
            float dy = (float(r) - mid)*div;
            float r = powf(1.0f - sqrtf(dx*dx + dy * dy), power);
            if (r < 0.0f) r = 0.0f;
            osg::Vec4 color = centerColor * r + bgColor * (1.0f - r);
            *ptr++ = (unsigned char)((color[0]) * 255.0f);
            *ptr++ = (unsigned char)((color[1]) * 255.0f);
            *ptr++ = (unsigned char)((color[2]) * 255.0f);
            *ptr++ = (unsigned char)((color[3]) * 255.0f);
        }
    }
    return image.release();
}

int main(int argc, char **argv)
{
    osg::Vec4 centerColor(1.0f, 1.0f, 0.0f, 1.0f);
    osg::Vec4 bgColor(0.0f, 0.0f, 0.0f, 1.0f);

    // 创建一个图像序列
    osg::ref_ptr<osg::ImageSequence> sequence = new osg::ImageSequence;
    sequence->addImage(createSpotLight(centerColor, bgColor, 64, 3.0f));
    sequence->addImage(createSpotLight(centerColor, bgColor, 64, 3.5f));
    sequence->addImage(createSpotLight(centerColor, bgColor, 64, 4.0f));
    sequence->addImage(createSpotLight(centerColor, bgColor, 64, 3.5f));

    // 将图像序列设置到纹理
    osg::ref_ptr<osg::Texture2D> texture = new osg::Texture2D;
    texture->setImage(sequence);

    osg::ref_ptr<osg::Geode> geode = new osg::Geode;
    geode->addDrawable(osg::createTexturedQuadGeometry(
        osg::Vec3(), osg::Vec3(1.0, 0.0, 0.0), osg::Vec3(0.0, 0.0, 1.0)));

    // 启用纹理
    geode->getOrCreateStateSet()->setTextureAttributeAndModes(
        0, texture, osg::StateAttribute::ON);

    sequence->setLength(0.5);  // 0.5秒
    sequence->play();

    osgViewer::Viewer viewer;
    viewer.setSceneData(geode);
    return viewer.run();
}
```

## 加载角色模型并播放动画

``` c++
#include <osgAnimation/BasicAnimationManager>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>
#include <iostream>

int main(int argc, char **argv)
{
    osg::ArgumentParser arguments(&argc, argv);

    std::string animationName;
    arguments.read("--animation", animationName);
    if (animationName.empty())
        animationName = "Idle_Main";

    bool listAll = false;
    if (arguments.read("--listall")) 
        listAll = true;

    osg::ref_ptr<osg::Node> model = osgDB::readNodeFile("bignathan.osg");
    if (!model) 
        return 1;

    osgAnimation::BasicAnimationManager* manager = 
        dynamic_cast<osgAnimation::BasicAnimationManager*>(model->getUpdateCallback());
    if (!manager) 
        return 1;

    const osgAnimation::AnimationList& animations = manager->getAnimationList();
    if (listAll) 
        std::cout << "**** Animations ****" << std::endl;

    for (unsigned int i = 0; i < animations.size(); ++i)
    {
        const std::string& name = animations[i]->getName();
        if (name == animationName)
            manager->playAnimation(animations[i]);

        if (listAll) 
            std::cout << name << std::endl;
    }

    if (listAll)
    {
        std::cout << "********************" << std::endl;
        return 0;
    }

    osgViewer::Viewer viewer;
    viewer.setSceneData(model);
    return viewer.run();
}
```

## 响应键盘事件

``` c++
#include <osg/MatrixTransform>
#include <osgDB/ReadFile>
#include <osgGA/GUIEventHandler>
#include <osgViewer/Viewer>

class ModelController : public osgGA::GUIEventHandler
{
public:
    ModelController(osg::MatrixTransform *node) : m_model(node)
    {}

    virtual bool handle(const osgGA::GUIEventAdapter &ea, osgGA::GUIActionAdapter &aa) override
    {
        if (!m_model)
            return false;

        osg::Matrix matrix = m_model->getMatrix();

        if (ea.getEventType() == osgGA::GUIEventAdapter::KEYDOWN) {
            switch (ea.getKey()) {
            case 'a':
            case 'A':
                matrix *= osg::Matrix::rotate(-0.1f, osg::Z_AXIS);
                break;
            case 'd':
            case 'D':
                matrix *= osg::Matrix::rotate(0.1f, osg::Z_AXIS);
                break;
            case 'w':
            case 'W':
                matrix *= osg::Matrix::rotate(-0.1f, osg::X_AXIS);
                break;
            case 's':
            case 'S':
                matrix *= osg::Matrix::rotate(0.1f, osg::X_AXIS);
                break;
            default:
                break;
            }

            m_model->setMatrix(matrix);
        }

        // 返回true表示事件已处理不需要继续传递事件，false表示继续传递事件到其他事件处理器
        return false;
    }

private:
    osg::ref_ptr<osg::MatrixTransform> m_model;
};

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Node> model = osgDB::readNodeFile("cessna.osg");

    osg::ref_ptr<osg::MatrixTransform> transform = new osg::MatrixTransform;
    transform->addChild(model);

    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(transform);

    osg::ref_ptr<ModelController> controller = new ModelController(transform);

    osgViewer::Viewer viewer;
    viewer.addEventHandler(controller);
    viewer.getCamera()->setViewMatrixAsLookAt(osg::Vec3(0.0f, -100.0f, 0.0f), osg::Vec3(), osg::Z_AXIS);
    viewer.getCamera()->setAllowEventFocus(false); // 禁用摄像机响应默认manipulator事件
    viewer.setSceneData(root.get());
    return viewer.run();
}
```

## 发送和处理用户事件

``` c++
#include <osg/Switch>
#include <osgDB/ReadFile>
#include <osgGA/GUIEventHandler>
#include <osgViewer/Viewer>
#include <iostream>

// 用户事件的自定义数据
struct TimerInfo : public osg::Referenced
{
    TimerInfo(unsigned int c) : count(c) {}
    unsigned int count;
};

class TimerHandler : public osgGA::GUIEventHandler
{
public:
    TimerHandler(osg::Switch *sw) : m_switch(sw), m_count(0) {}
    virtual bool handle(const osgGA::GUIEventAdapter &ea, osgGA::GUIActionAdapter &aa) override
    {
        switch (ea.getEventType()) {
        case osgGA::GUIEventAdapter::FRAME:
            if (m_count % 100 == 0) {
                osgViewer::Viewer* viewer = dynamic_cast<osgViewer::Viewer*>(&aa);
                if (viewer) {
                    // 发送用户事件
                    viewer->getEventQueue()->userEvent(new TimerInfo(m_count));
                }
            }
            m_count++;
            break;
        case osgGA::GUIEventAdapter::USER:
            // 处理用户事件
            if (m_switch.valid()) {
                // 获取用户事件自定义数据
                const TimerInfo* ti = dynamic_cast<const TimerInfo*>(ea.getUserData());
                std::cout << "Timer event at: " << ti->count << std::endl;

                m_switch->setValue(0, !m_switch->getValue(0));
                m_switch->setValue(1, !m_switch->getValue(1));
            }
            break;
        default:
            break;
        }
        return false;
    }

protected:
    osg::ref_ptr<osg::Switch> m_switch;
    unsigned int m_count;
};

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Node> model1 = osgDB::readNodeFile("cessna.osg");
    osg::ref_ptr<osg::Node> model2 = osgDB::readNodeFile("cessnafire.osg");
    osg::ref_ptr<osg::Switch> root = new osg::Switch;
    root->addChild(model1, false);
    root->addChild(model2, true);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    viewer.addEventHandler(new TimerHandler(root));
    return viewer.run();
}
```

## 拾取

``` c++
#include <osg/MatrixTransform>
#include <osg/ShapeDrawable>
#include <osg/PolygonMode>
#include <osgDB/ReadFile>
#include <osgUtil/LineSegmentIntersector>
#include <osgViewer/Viewer>

class PickHandler : public osgGA::GUIEventHandler
{
public:
    osg::Node* getOrCreateSelectionBox()
    {
        if (!m_selectionBox)
        {
            osg::ref_ptr<osg::Geode> geode = new osg::Geode;
            geode->addDrawable(new osg::ShapeDrawable(new osg::Box(osg::Vec3(), 1.0f)));

            m_selectionBox = new osg::MatrixTransform;
            m_selectionBox->setNodeMask(0x1);
            m_selectionBox->addChild(geode);

            osg::StateSet* ss = m_selectionBox->getOrCreateStateSet();
            ss->setMode(GL_LIGHTING, osg::StateAttribute::OFF);
            ss->setAttributeAndModes(new osg::PolygonMode(
                osg::PolygonMode::FRONT_AND_BACK, osg::PolygonMode::LINE));
        }

        return m_selectionBox.get();
    }

    virtual bool handle(const osgGA::GUIEventAdapter &ea, osgGA::GUIActionAdapter &aa)
    {
        if (ea.getEventType() != osgGA::GUIEventAdapter::RELEASE
            || ea.getButton() != osgGA::GUIEventAdapter::LEFT_MOUSE_BUTTON
            || !(ea.getModKeyMask() & osgGA::GUIEventAdapter::MODKEY_CTRL)) {
            return false;
        }

        osgViewer::Viewer* viewer = dynamic_cast<osgViewer::Viewer*>(&aa);
        if (viewer) {
            osg::ref_ptr<osgUtil::LineSegmentIntersector> intersector =
                new osgUtil::LineSegmentIntersector(osgUtil::Intersector::WINDOW, ea.getX(), ea.getY());

            // 执行相交测试
            osgUtil::IntersectionVisitor iv(intersector);
            iv.setTraversalMask(~0x1);
            viewer->getCamera()->accept(iv);

            if (intersector->containsIntersections()) {
                const osgUtil::LineSegmentIntersector::Intersection &result = *(intersector->getIntersections().begin());

                // 修改包围盒大小匹配拾取的对象
                osg::BoundingBox bb = result.drawable->getBoundingBox();
                osg::Vec3 worldCenter = bb.center() * osg::computeLocalToWorld(result.nodePath);
                m_selectionBox->setMatrix(
                    osg::Matrix::scale(bb.xMax() - bb.xMin(), bb.yMax() - bb.yMin(), bb.zMax() - bb.zMin()) *
                    osg::Matrix::translate(worldCenter));
            }
        }

        return false;
    }

protected:
    osg::ref_ptr<osg::MatrixTransform> m_selectionBox;
};

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Node> model1 = osgDB::readNodeFile("cessna.osg");
    osg::ref_ptr<osg::Node> model2 = osgDB::readNodeFile("cow.osg");

    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(model1);
    root->addChild(model2);

    osg::ref_ptr<PickHandler> picker = new PickHandler;
    root->addChild(picker->getOrCreateSelectionBox());

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    viewer.addEventHandler(picker);
    return viewer.run();
}
```

## 配置 GraphicsContext

``` c++
#include <osg/GraphicsContext>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::GraphicsContext::Traits> traits = new osg::GraphicsContext::Traits;
    traits->x = 50;
    traits->y = 50;
    traits->width = 800;
    traits->height = 600;
    traits->windowDecoration = true; // 是否带标题栏和边框
    traits->doubleBuffer = true;     // 双缓冲
    traits->samples = 4;             // 多重采样

    osg::ref_ptr<osg::GraphicsContext> gc =
        osg::GraphicsContext::createGraphicsContext(traits);

    osg::ref_ptr<osg::Camera> camera = new osg::Camera;
    camera->setGraphicsContext(gc);
    camera->setViewport(new osg::Viewport(0, 0, traits->width, traits->height));
    camera->setClearMask(GL_DEPTH_BUFFER_BIT | GL_COLOR_BUFFER_BIT);
    camera->setClearColor(osg::Vec4f(0.2f, 0.2f, 0.4f, 1.0f));
    camera->setProjectionMatrixAsPerspective(
        30.0f, (double)traits->width / (double)traits->height, 1.0, 1000.0);

    osg::ref_ptr<osg::Node> root = osgDB::readNodeFile("cessna.osg");

    osgViewer::Viewer viewer;
    viewer.setCamera(camera);
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 广告牌

``` c++
#include <osg/Billboard>
#include <osg/Texture2D>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

osg::Geometry* createQuad()
{
    osg::ref_ptr<osg::Texture2D> texture = new osg::Texture2D;
    osg::ref_ptr<osg::Image> image = osgDB::readImageFile("Images/osg256.png");
    texture->setImage(image);

    osg::ref_ptr<osg::Geometry> quad = osg::createTexturedQuadGeometry(
        osg::Vec3(-0.5f, 0.0f, -0.5f), osg::Vec3(1.0f, 0.0f, 0.0f), osg::Vec3(0.0f, 0.0f, 1.0f));

    osg::StateSet* ss = quad->getOrCreateStateSet();
    ss->setTextureAttributeAndModes(0, texture);
    return quad.release();
}

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Billboard> geode = new osg::Billboard;
    geode->setMode(osg::Billboard::POINT_ROT_EYE);

    osg::Geometry* quad = createQuad();
    for (unsigned int i = 0; i < 10; ++i)
    {
        float id = (float)i;
        geode->addDrawable(quad, osg::Vec3(-2.5f + 0.2f * id, id, 0.0f));
        geode->addDrawable(quad, osg::Vec3(2.5f - 0.2f * id, id, 0.0f));
    }

    osg::StateSet* ss = geode->getOrCreateStateSet();
    ss->setRenderingHint(osg::StateSet::TRANSPARENT_BIN);

    osgViewer::Viewer viewer;
    viewer.setSceneData(geode);
    return viewer.run();
}
```

## 显示文本

``` c++
#include <osg/Camera>
#include <osgDB/ReadFile>
#include <osgText/Font>
#include <osgText/Text>
#include <osgViewer/Viewer>

osg::ref_ptr<osgText::Font> g_font = osgText::readFontFile("fonts/arial.ttf");

osg::Camera* createHUDCamera(double left, double right, double bottom, double top)
{
    osg::ref_ptr<osg::Camera> camera = new osg::Camera;
    camera->setReferenceFrame(osg::Transform::ABSOLUTE_RF);
    camera->setClearMask(GL_DEPTH_BUFFER_BIT);
    camera->setRenderOrder(osg::Camera::POST_RENDER);
    camera->setAllowEventFocus(false);
    camera->setProjectionMatrix(osg::Matrix::ortho2D(left, right, bottom, top));
    return camera.release();
}

osgText::Text* createText(const osg::Vec3& pos, const std::string& content, float size)
{
    osg::ref_ptr<osgText::Text> text = new osgText::Text;
    text->setFont(g_font);
    text->setCharacterSize(size);
    text->setAxisAlignment(osgText::TextBase::XY_PLANE);
    text->setPosition(pos);
    text->setText(content);
    return text.release();
}

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Geode> textGeode = new osg::Geode;
    textGeode->addDrawable(createText(
        osg::Vec3(150.0f, 500.0f, 0.0f), 
        "The Cessna monoplane", 
        20.0f));
    textGeode->addDrawable(createText(
        osg::Vec3(150.0f, 450.0f, 0.0f), 
        "Six-seat, low-wing and twin-engined", 
        15.0f));

    osg::Camera* camera = createHUDCamera(0, 1024, 0, 768);
    camera->addChild(textGeode);
    camera->getOrCreateStateSet()->setMode(GL_LIGHTING, osg::StateAttribute::OFF);

    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(osgDB::readNodeFile("cessna.osg"));
    root->addChild(camera);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 3D文本

``` c++
#include <osg/MatrixTransform>
#include <osgDB/ReadFile>
#include <osgText/Font3D>
#include <osgText/Text3D>
#include <osgViewer/Viewer>

osg::ref_ptr<osgText::Font3D> g_font3D = osgText::readFont3DFile("fonts/arial.ttf");

osgText::Text3D* createText3D(
    const osg::Vec3& pos, const std::string& content, float size, float depth)
{
    osg::ref_ptr<osgText::Text3D> text = new osgText::Text3D;
    text->setFont(g_font3D);
    text->setCharacterSize(size);
    text->setCharacterDepth(depth);
    text->setAxisAlignment(osgText::TextBase::XZ_PLANE);
    text->setPosition(pos);
    text->setText(content);
    return text.release();
}

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::Geode> textGeode = new osg::Geode;
    textGeode->addDrawable(createText3D(osg::Vec3(), "The Cessna", 20.0f, 10.0f));

    osg::ref_ptr<osg::MatrixTransform> textNode = new osg::MatrixTransform;
    textNode->setMatrix(osg::Matrix::translate(0.0f, 0.0f, 10.0f));
    textNode->addChild(textGeode);

    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(osgDB::readNodeFile("cessna.osg"));
    root->addChild(textNode);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 粒子动画

``` c++
#include <osg/MatrixTransform>
#include <osg/Point>
#include <osg/PointSprite>
#include <osg/Texture2D>
#include <osg/BlendFunc>
#include <osgDB/ReadFile>
#include <osgGA/StateSetManipulator>
#include <osgParticle/ParticleSystem>
#include <osgParticle/ParticleSystemUpdater>
#include <osgParticle/ModularEmitter>
#include <osgParticle/ModularProgram>
#include <osgParticle/AccelOperator>
#include <osgViewer/ViewerEventHandlers>
#include <osgViewer/Viewer>

osgParticle::ParticleSystem* createParticleSystem(osg::Group* parent)
{
    osg::ref_ptr<osgParticle::ParticleSystem> ps = new osgParticle::ParticleSystem;
    ps->getDefaultParticleTemplate().setShape(osgParticle::Particle::POINT);

    osg::ref_ptr<osg::BlendFunc> blendFunc = new osg::BlendFunc;
    blendFunc->setFunction(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);

    osg::ref_ptr<osg::Texture2D> texture = new osg::Texture2D;
    texture->setImage(osgDB::readImageFile("Images/smoke.rgb"));

    osg::StateSet* ss = ps->getOrCreateStateSet();
    ss->setAttributeAndModes(blendFunc);          // 设置混合模式
    ss->setTextureAttributeAndModes(0, texture);  // 设置纹理对象
    ss->setAttribute(new osg::Point(20.0f));
    ss->setTextureAttributeAndModes(0, new osg::PointSprite);
    ss->setMode(GL_LIGHTING, osg::StateAttribute::OFF);    // 禁用光照
    ss->setRenderingHint(osg::StateSet::TRANSPARENT_BIN);  // 透明

    osg::ref_ptr<osgParticle::RandomRateCounter> rrc = new osgParticle::RandomRateCounter;
    rrc->setRateRange(500, 800);

    osg::ref_ptr<osgParticle::ModularEmitter> emitter = new osgParticle::ModularEmitter;
    emitter->setParticleSystem(ps);
    emitter->setCounter(rrc);

    osg::ref_ptr<osgParticle::AccelOperator> accel = new osgParticle::AccelOperator;
    accel->setToGravity();

    osg::ref_ptr<osgParticle::ModularProgram> program = new osgParticle::ModularProgram;
    program->setParticleSystem(ps);
    program->addOperator(accel);

    osg::ref_ptr<osg::Geode> geode = new osg::Geode;
    geode->addDrawable(ps);
    parent->addChild(emitter);
    parent->addChild(program);
    parent->addChild(geode);
    return ps;
}

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::MatrixTransform> mt = new osg::MatrixTransform;
    mt->setMatrix(osg::Matrix::translate(1.0f, 0.0f, 0.0f));

    osgParticle::ParticleSystem* ps = createParticleSystem(mt);

    osg::ref_ptr<osgParticle::ParticleSystemUpdater> updater = new osgParticle::ParticleSystemUpdater;
    updater->addParticleSystem(ps);

    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(updater);
    root->addChild(mt);
    root->addChild(osgDB::readNodeFile("axes.osgt"));

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 粒子动画

``` c++
#include <osg/MatrixTransform>
#include <osg/Point>
#include <osg/PointSprite>
#include <osg/Texture2D>
#include <osg/BlendFunc>
#include <osgDB/ReadFile>
#include <osgGA/StateSetManipulator>
#include <osgParticle/ParticleSystem>
#include <osgParticle/ParticleSystemUpdater>
#include <osgParticle/ModularEmitter>
#include <osgParticle/ModularProgram>
#include <osgParticle/AccelOperator>
#include <osgViewer/ViewerEventHandlers>
#include <osgViewer/Viewer>

osgParticle::ParticleSystem* createParticleSystem(osg::Group* parent)
{
    osg::ref_ptr<osgParticle::ParticleSystem> ps = new osgParticle::ParticleSystem;
    ps->getDefaultParticleTemplate().setShape(osgParticle::Particle::POINT);

    osg::ref_ptr<osg::BlendFunc> blendFunc = new osg::BlendFunc;
    blendFunc->setFunction(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);

    osg::ref_ptr<osg::Texture2D> texture = new osg::Texture2D;
    texture->setImage(osgDB::readImageFile("Images/smoke.rgb"));

    osg::StateSet* ss = ps->getOrCreateStateSet();
    ss->setAttributeAndModes(blendFunc);          // 设置混合模式
    ss->setTextureAttributeAndModes(0, texture);  // 设置纹理对象
    ss->setAttribute(new osg::Point(20.0f));
    ss->setTextureAttributeAndModes(0, new osg::PointSprite);
    ss->setMode(GL_LIGHTING, osg::StateAttribute::OFF);    // 禁用光照
    ss->setRenderingHint(osg::StateSet::TRANSPARENT_BIN);  // 透明

    // 各组件包含关系
    // ModularEmitter
    //     Counter
    //     Placer
    //     Shooter
    // ModularProgram
    //     Operator
    // Geode
    //     ParticleSystem
    // ParticleSystemUpdater

    // 决定创建粒子的个数
    osg::ref_ptr<osgParticle::RandomRateCounter> rrc = new osgParticle::RandomRateCounter;
    rrc->setRateRange(500, 800);

    // 粒子发射器
    osg::ref_ptr<osgParticle::ModularEmitter> emitter = new osgParticle::ModularEmitter;
    emitter->setParticleSystem(ps);
    emitter->setCounter(rrc);

    // 控制粒子的速度
    osg::ref_ptr<osgParticle::AccelOperator> accel = new osgParticle::AccelOperator;
    accel->setToGravity();

    // 包含多个Operator
    osg::ref_ptr<osgParticle::ModularProgram> program = new osgParticle::ModularProgram;
    program->setParticleSystem(ps);
    program->addOperator(accel);

    osg::ref_ptr<osg::Geode> geode = new osg::Geode;
    geode->addDrawable(ps);
    parent->addChild(emitter);
    parent->addChild(program);
    parent->addChild(geode);
    return ps;
}

int main(int argc, char **argv)
{
    osg::ref_ptr<osg::MatrixTransform> mt = new osg::MatrixTransform;
    mt->setMatrix(osg::Matrix::translate(1.0f, 0.0f, 0.0f));

    osgParticle::ParticleSystem* ps = createParticleSystem(mt);

    osg::ref_ptr<osgParticle::ParticleSystemUpdater> updater = new osgParticle::ParticleSystemUpdater;
    updater->addParticleSystem(ps);

    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(updater);
    root->addChild(mt);
    root->addChild(osgDB::readNodeFile("axes.osgt"));

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 阴影

``` c++
#include <osg/AnimationPath>
#include <osg/MatrixTransform>
#include <osgDB/ReadFile>
#include <osgShadow/ShadowedScene>
#include <osgShadow/ShadowMap>
#include <osgViewer/Viewer>
osg::AnimationPath* createAnimationPath(float radius, float time) 
{
    osg::ref_ptr<osg::AnimationPath> path = new osg::AnimationPath;
    path->setLoopMode(osg::AnimationPath::LOOP);

    unsigned int numSamples = 32;
    float delta_yaw = 2.0f * osg::PI / ((float)numSamples - 1.0f);
    float delta_time = time / (float)numSamples;

    for (unsigned int i = 0; i < numSamples; ++i) {
        float yaw = delta_yaw * (float)i;
        osg::Vec3 pos(sinf(yaw)*radius, cosf(yaw)*radius, 0.0f);
        osg::Quat rot(-yaw, osg::Z_AXIS);
        path->insert(delta_time * (float)i, osg::AnimationPath::ControlPoint(pos, rot));
    }

    return path.release();
}

int main(int argc, char **argv)
{
    unsigned int rcvShadowMask = 0x1;
    unsigned int castShadowMask = 0x2;

    osg::ref_ptr<osg::MatrixTransform> groundNode = new osg::MatrixTransform;
    groundNode->addChild(osgDB::readNodeFile("lz.osg"));
    groundNode->setMatrix(osg::Matrix::translate(0.0f, 0.0f, -200.0f));
    groundNode->setNodeMask(rcvShadowMask);

    osg::ref_ptr<osg::MatrixTransform> cessnaNode = new osg::MatrixTransform;
    cessnaNode->addChild(osgDB::readNodeFile("cessna.osg.0,0,90.rot"));
    cessnaNode->setNodeMask(castShadowMask);

    osg::ref_ptr<osg::AnimationPathCallback> apcb = new osg::AnimationPathCallback;
    apcb->setAnimationPath(createAnimationPath(50.0f, 6.0f));
    cessnaNode->setUpdateCallback(apcb);

    osg::ref_ptr<osg::MatrixTransform> truckNode = new osg::MatrixTransform;
    truckNode->addChild(osgDB::readNodeFile("dumptruck.osg"));
    truckNode->setMatrix(osg::Matrix::translate(0.0f, 0.0f, -100.0f));
    truckNode->setNodeMask(rcvShadowMask | castShadowMask);

    osg::ref_ptr<osg::LightSource> source = new osg::LightSource;
    source->getLight()->setPosition(osg::Vec4(4.0, 4.0, 10.0, 0.0));
    source->getLight()->setAmbient(osg::Vec4(0.2, 0.2, 0.2, 1.0));
    source->getLight()->setDiffuse(osg::Vec4(0.8, 0.8, 0.8, 1.0));

    osg::ref_ptr<osgShadow::ShadowMap> sm = new osgShadow::ShadowMap;
    sm->setLight(source);
    sm->setTextureSize(osg::Vec2s(1024, 1024));
    sm->setTextureUnit(1);

    osg::ref_ptr<osgShadow::ShadowedScene> root = new osgShadow::ShadowedScene;
    root->setShadowTechnique(sm);
    root->setReceivesShadowTraversalMask(rcvShadowMask);
    root->setCastsShadowTraversalMask(castShadowMask);
    root->addChild(groundNode);
    root->addChild(cessnaNode);
    root->addChild(truckNode);
    root->addChild(source);

    osgViewer::Viewer viewer;
    viewer.setSceneData(root);
    return viewer.run();
}
```

## 地形四叉树

``` c++
#include <osg/ShapeDrawable>
#include <osg/PagedLOD>
#include <osgDB/WriteFile>
#include <sstream>

float* g_data = NULL;
float g_dx = 1.0f;
float g_dy = 1.0f;
unsigned int g_minCols = 64;
unsigned int g_minRows = 64;
unsigned int g_numCols = 1024;
unsigned int g_numRows = 1024;

#define RAND(min, max)  ((min) + (float)rand()/(RAND_MAX+1) * ((max)-(min)))

void createMassiveData()
{
    g_data = new float[g_numCols * g_numRows];
    for (unsigned int i = 0; i < g_numRows; ++i)
    {
        for (unsigned int j = 0; j < g_numCols; ++j)
            g_data[i*g_numCols + j] = RAND(0.5f, 0.0f);
    }
}

float getOneData(unsigned int c, unsigned int r)
{
    return g_data[osg::minimum(r, g_numRows - 1) * g_numCols + osg::minimum(c, g_numCols - 1)];
}

std::string createFileName(unsigned int lv, unsigned int x, unsigned int y)
{
    std::stringstream sstream;
    sstream << "quadtree_L" << lv << "_X" << x << "_Y" << y <<
        ".osg";
    return sstream.str();
}

osg::Node* outputSubScene(unsigned int lv, unsigned int x, unsigned int y, const osg::Vec4& color)
{
    unsigned int numInUnitCol = g_numCols / (int)powf(2.0f, (float)lv);
    unsigned int numInUnitRow = g_numRows / (int)powf(2.0f, (float)lv);
    unsigned int xDataStart = x * numInUnitCol;
    unsigned int xDataEnd = (x + 1) * numInUnitCol;
    unsigned int yDataStart = y * numInUnitRow;
    unsigned int  yDataEnd = (y + 1) * numInUnitRow;

    bool stopAtLeafNode = false;
    osg::ref_ptr<osg::HeightField> grid = new osg::HeightField;
    grid->setSkirtHeight(1.0f);
    grid->setOrigin(osg::Vec3(g_dx*(float)xDataStart, g_dy*(float)yDataStart, 0.0f));

    if (xDataEnd - xDataStart <= g_minCols && yDataEnd - yDataStart <= g_minRows)
    {
        grid->allocate(xDataEnd - xDataStart + 1, yDataEnd - yDataStart + 1);
        grid->setXInterval(g_dx);
        grid->setYInterval(g_dy);

        for (unsigned int i = yDataStart; i <= yDataEnd; ++i)
        {
            for (unsigned int j = xDataStart; j <= xDataEnd; ++j)
            {
                grid->setHeight(j - xDataStart, i - yDataStart, getOneData(j, i));
            }
        }
        stopAtLeafNode = true;
    }
    else
    {
        unsigned int jStep = (unsigned int)ceilf((float)(xDataEnd - xDataStart) / (float)g_minCols);
        unsigned int iStep = (unsigned int)ceilf((float)(yDataEnd - yDataStart) / (float)g_minRows);
        grid->allocate(g_minCols + 1, g_minRows + 1);
        grid->setXInterval(g_dx * jStep);
        grid->setYInterval(g_dy * iStep);

        for (unsigned int i = yDataStart, ii = 0; i <= yDataEnd; i += iStep, ++ii)
        {
            for (unsigned int j = xDataStart, jj = 0; j <= xDataEnd; j += jStep, ++jj)
            {
                grid->setHeight(jj, ii, getOneData(j, i));
            }
        }
    }
    osg::ref_ptr<osg::ShapeDrawable> shape = new osg::ShapeDrawable(grid.get());
    shape->setColor(color);

    osg::ref_ptr<osg::Geode> geode = new osg::Geode;
    geode->addDrawable(shape.get());

    if (stopAtLeafNode) 
        return geode.release();


    osg::ref_ptr<osg::Group> group = new osg::Group;
    group->addChild(outputSubScene(lv + 1, x * 2, y * 2, osg::Vec4(1.0f, 0.0f, 0.0f, 1.0f)));
    group->addChild(outputSubScene(lv + 1, x * 2, y * 2 + 1, osg::Vec4(0.0f, 1.0f, 0.0f, 1.0f)));
    group->addChild(outputSubScene(lv + 1, x * 2 + 1, y * 2 + 1, osg::Vec4(0.0f, 0.0f, 1.0f, 1.0f)));
    group->addChild(outputSubScene(lv + 1, x * 2 + 1, y * 2, osg::Vec4(1.0f, 1.0f, 0.0f, 1.0f)));

    osg::ref_ptr<osg::PagedLOD> plod = new osg::PagedLOD;
    std::string filename = createFileName(lv, x, y);
    plod->insertChild(0, geode.get());
    plod->setFileName(1, filename);
    osgDB::writeNodeFile(*group, filename);

    plod->setCenterMode(osg::PagedLOD::USER_DEFINED_CENTER);
    plod->setCenter(geode->getBound().center());
    plod->setRadius(geode->getBound().radius());
    float cutoff = geode->getBound().radius() * 5.0f;
    plod->setRange(0, cutoff, FLT_MAX);
    plod->setRange(1, 0.0f, cutoff);
    return plod.release();
}

int main(int argc, char **argv)
{
    createMassiveData();

    osg::ref_ptr<osg::Group> root = new osg::Group;
    root->addChild(outputSubScene(0, 0, 0, osg::Vec4(1.0f, 1.0f, 1.0f, 1.0f)));

    osgDB::writeNodeFile(*root, "quadtree.osg");
    delete g_data;

    return 0;
}
```