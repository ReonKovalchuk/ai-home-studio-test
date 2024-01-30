<template>
  <div class="container">
    <canvas
      id="canvas"
      width="600"
      height="600"
      ref="canvasRef"
      style="border: 1px solid #ccc"
    ></canvas>
    <label for="" class="input-group">
      <input
        v-model="length"
        type="number"
        min="5"
        max="20000"
        step="10"
        @input="changePolylineLength"
      />
      см - пока не меняет размер линии</label
    >
  </div>
</template>

<script setup>
import { fabric } from "fabric";
import { onMounted, ref } from "vue";
const length = ref(2000);
var canvas;

onMounted(() => {
  canvas = new fabric.Canvas("canvas");
  // create a polygon/polyline object
  var points = [
    {
      x: 0,
      y: 5,
    },
    {
      x: length.value,
      y: 5,
    },
  ];

  var polyline = new fabric.Polyline(points, {
    left: canvas.getWidth() / 2,
    top: canvas.getHeight() / 2,
    fill: "transparent",
    strokeWidth: 100,
    stroke: "#dedede",
    scaleX: 0.1,
    scaleY: 0.1,
    objectCaching: false,
    transparentCorners: false,
    edit: true,
    hasBorders: true,
    cornerStyle: "circle",
    cornerColor: "rgba(0,0,255,0.5)",
    controls: points.reduce(function (acc, point, index) {
      acc["p" + index] = new fabric.Control({
        positionHandler: polygonPositionHandler,
        actionHandler: anchorWrapper(index > 0 ? index - 1 : 1, actionHandler),
        actionName: "modifyPolygon",
        pointIndex: index,
      });
      return acc;
    }, {}),
  });
  // canvas.viewportTransform = [0.7, 0, 0, 0.7, -50, 50];
  canvas.add(polyline);
  canvas.setActiveObject(polyline);
  canvas.requestRenderAll();
});

//define function that can change lengh of polyline based on input
function changePolylineLength() {}

// define a function that can locate the controls.
// this function will be used both for drawing and for interaction.
function polygonPositionHandler(dim, finalMatrix, fabricObject) {
  var x = fabricObject.points[this.pointIndex].x - fabricObject.pathOffset.x,
    y = fabricObject.points[this.pointIndex].y - fabricObject.pathOffset.y;
  return fabric.util.transformPoint(
    { x: x, y: y },
    fabric.util.multiplyTransformMatrices(
      fabricObject.canvas.viewportTransform,
      fabricObject.calcTransformMatrix()
    )
  );
}

function getObjectSizeWithStroke(object) {
  var stroke = new fabric.Point(
    object.strokeUniform ? 1 / object.scaleX : 1,
    object.strokeUniform ? 1 / object.scaleY : 1
  ).multiply(object.strokeWidth);
  return new fabric.Point(object.width + stroke.x, object.height + stroke.y);
}

// define a function that will define what the control does
// this function will be called on every mouse move after a control has been
// clicked and is being dragged.
// The function receive as argument the mouse event, the current trasnform object
// and the current position in canvas coordinate
// transform.target is a reference to the current object being transformed,
function actionHandler(eventData, transform, x, y) {
  var polygon = transform.target,
    currentControl = polygon.controls[polygon.__corner],
    mouseLocalPosition = polygon.toLocalPoint(
      new fabric.Point(x, y),
      "center",
      "center"
    ),
    polygonBaseSize = getObjectSizeWithStroke(polygon),
    size = polygon._getTransformedDimensions(0, 0),
    finalPointPosition = {
      x:
        (mouseLocalPosition.x * polygonBaseSize.x) / size.x +
        polygon.pathOffset.x,
      y:
        (mouseLocalPosition.y * polygonBaseSize.y) / size.y +
        polygon.pathOffset.y,
    };
  polygon.points[currentControl.pointIndex] = finalPointPosition;

  //add functionality to change length input on change
  const currentDimentions = polygon.getBoundingRect(true);
  length.value = Math.round(
    Math.sqrt(
      (currentDimentions.width * 10) ** 2 + (currentDimentions.height * 10) ** 2
    )
  );

  return true;
}

// define a function that can keep the polygon in the same position when we change its
// width/height/top/left.
function anchorWrapper(anchorIndex, fn) {
  return function (eventData, transform, x, y) {
    var fabricObject = transform.target,
      absolutePoint = fabric.util.transformPoint(
        {
          x: fabricObject.points[anchorIndex].x - fabricObject.pathOffset.x,
          y: fabricObject.points[anchorIndex].y - fabricObject.pathOffset.y,
        },
        fabricObject.calcTransformMatrix()
      ),
      actionPerformed = fn(eventData, transform, x, y),
      // eslint-disable-next-line no-unused-vars
      newDim = fabricObject._setPositionDimensions({}),
      polygonBaseSize = getObjectSizeWithStroke(fabricObject),
      newX =
        (fabricObject.points[anchorIndex].x - fabricObject.pathOffset.x) /
        polygonBaseSize.x,
      newY =
        (fabricObject.points[anchorIndex].y - fabricObject.pathOffset.y) /
        polygonBaseSize.y;
    fabricObject.setPositionByOrigin(absolutePoint, newX + 0.5, newY + 0.5);
    return actionPerformed;
  };
}
</script>

<style>
.container {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}
input {
  min-width: 20px;
  max-width: 60px;
}
</style>
