# IFC models loading in CesiumJS

In this meet I will present you a small wrapper library to silently convert an IFC file to gLTF, directly in the browser.

## IFC?

IFC stands for "Industry Foundation Classes". It is a used to store 3d models, like buildings.
It is related to "Building Information Modeling (BIM)".

## gLTF?

glTF stands for "Graphics Library Transmission Format". It is used to store 3d models, like buildings.
It was created by Cesium and standardised by Khonos.

## IFC ?== glTF

No, IFC is used for editing while glTF was designed for optimized transmission and rendering.

## geoblocks/ifc-gltf (frontend)

This [geoblock](https://github.com/geoblocks/ifc-gltf) allows to convert an IFC file directly in the browser.

```js
const element = document.createElement('div');
const ifcUrl = "https://thatopen.github.io/engine_components/resources/small.ifc";
const {glb, coordinationMatrix, metadata} = await ifcToGLTF({
  element: element,
  url: ifcUrl,
})
```

Et voilà, you receive:

- `glb`, the binary version of a [glTF](https://www.khronos.org/gltf/)
- `coordinationMatrix`, the transformation matrix to position the model (unfortunately, usually the identity matrix);
- `metadata`, some metadata associated with the IFC.

You can then load that model into [CesiumJS](https://github.com/CesiumGS/cesium/) or any other 3D engine supporting glTF.

## Integration inside NGV

[NGV](https://github.com/geoblocks/ngv) is our framework for building 3D applications.

You can add an IFC "layer" with:

```ts
export interface INGVIFC {
  type: 'ifc';
  url: string;
  options?: {
    modelOptions: Omit<INGVCesiumModelConfig['options'], 'url'>;
  };
  position: [number, number];
  height: number;
  rotation: number;
}
```

Demo: https://ngv.labs.camptocamp.com/src/apps/permits/index.html

## Cesium Ion (backend)

Cesium is currently working on IFC support in there coming AECO support.
https://cesium.com/blog/2024/09/03/join-the-cesium-aeco-tech-preview-program/

We are part of the program and will test this backend solution.