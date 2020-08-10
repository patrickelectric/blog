---
title: "QT3D, Lines and AtCore"
header:
    teaser: /assets/QT3DLines/cube.png
tags:
    - AtCore
    - Atelier
    - KDE
    - QT3D
---

After some time working with Qt3D, now Atelier project is one step closer to have a 3D viewer from the GCode and a realtime draw of printer work.

![3D draw with lines](/assets/QT3DLines/atcore.gif)

To know how Qt3D works, you can take a look in this [overview](https://doc.qt.io/qt-5/qt3d-overview.html). Where, working with 3D lines was a bit problematic, but I hope to help someone with this job.

# Rendering

To render, we need our mesh and the material. This simple cube have something more than 1300 lines, and if you are thinking about it, you can can handle big models like this [Dragon Head](https://www.thingiverse.com/thing:51415) with around 706000 lines in 60fps.

![dragon](/assets/QT3DLines/dragon.png)

### Mesh

It's done with a C++ code, where a 4D vector is used to perform the buffer population, the 4D is because a 3D printer have 4 dimensions in a actuation space, (X, Y, Z and E) where E is the extruder that move the filament to perform the extrusion.

```cpp
LineMeshGeometry::LineMeshGeometry(QList<QVector4D> vertices, Qt3DCore::QNode *parent) :
    Qt3DRender::QGeometry(parent)
    , _positionAttribute(new Qt3DRender::QAttribute(this))
    , _vertexBuffer(new Qt3DRender::QBuffer(Qt3DRender::QBuffer::VertexBuffer, this))
{
    QByteArray vertexBufferData;
    vertexBufferData.resize(vertices.size() * 3 * sizeof(float));
    float *rawVertexArray = reinterpret_cast<float *>(vertexBufferData.data());
    int idx = 0;
    for (const auto &v : vertices) {
        rawVertexArray[idx++] = v.x();
        rawVertexArray[idx++] = v.y();
        rawVertexArray[idx++] = v.z();
        _vertices.append(v.toVector3D());
    }

    _vertexBuffer->setData(vertexBufferData);

    _positionAttribute->setAttributeType(Qt3DRender::QAttribute::VertexAttribute);
    _positionAttribute->setBuffer(_vertexBuffer);
    _positionAttribute->setDataType(Qt3DRender::QAttribute::Float);
    _positionAttribute->setDataSize(3);
    _positionAttribute->setName(Qt3DRender::QAttribute::defaultPositionAttributeName());

    addAttribute(_positionAttribute);
}

int LineMeshGeometry::vertexCount()
{
    return _vertices.size();
}
```

With this done, it's possible to easy populate it with a simple class.

```cpp
LineMesh::LineMesh(Qt3DCore::QNode *parent) :
    Qt3DRender::QGeometryRenderer(parent)
    , _lineMeshGeo(nullptr)
{
    setInstanceCount(1);
    setIndexOffset(0);
    setFirstInstance(0);
    // This will allow the line visualization
    setPrimitiveType(Qt3DRender::QGeometryRenderer::LineStrip);

    // This will be visualized in qml
    qRegisterMetaType<QList<QVector4D> >("QList<QVector4D>");
    auto gcode = new GcodeTo3D();
    connect(gcode, &GcodeTo3D::posFinished, this, &LineMesh::posUpdate);
}

void LineMesh::posUpdate(QList<QVector4D> pos)
{
    _vertices = pos;
    _lineMeshGeo = new LineMeshGeometry(_vertices, this);
    setVertexCount(_lineMeshGeo->vertexCount());
    setGeometry(_lineMeshGeo);
    emit finished();
}
```

### Material and Entity
The material used in the first GIF was [PhongMaterial](https://doc.qt.io/qt-5/qml-qt3d-extras-phongmaterial.html), this is simple done with:

```qml
PhongMaterial {
  id: lineMaterial
  ambient: "darkGreen"
}
```

And in the end, the creation of an Entity.

```qml
LineMesh {
  id: lineMesh
  enabled: true
}
```

```qml
Entity {
  id: lineEntity
  components: [ lineMesh, lineMaterial ]
}
```

In the next week we plan to finish our simple 3D viwer and add it in AtCore test GUI.\\
For more info about [AtCore and Atelier click here !](https://github.com/kde/atcore)

![Cube 2](/assets/QT3DLines/cube2.png)