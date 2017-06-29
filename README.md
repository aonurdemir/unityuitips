# Unity UI Tips
In general, the UI Text component's Best Fit setting should never be used. Since,
“Best Fit” dynamically adjusts the size of a font to the largest integer point size which can be displayed within a Text component’s bounding box without overflow, clamped to a configurable minimum/maximum point size. However, because Unity renders a distinct glyph into the font atlas for each distinct size of character being displayed, use of Best Fit will rapidly overwhelm the atlas with many different glyph sizes.

If Canvas.BuildBatch or Canvas::UpdateBatches seems to be using an excessive amount of CPU time, then the likely problem is an excessive number of Canvas Renderer components on a single Canvas. See the Splitting Canvases section of the Canvas chapter.

If an excessive amount of time is spent drawing the UI on the GPU, and the frame debugger indicates that the fragment shader pipeline is the bottleneck, then the UI is likely exceeding the pixel fill rate which the GPU is capable of. The most likely cause is excessive UI overdraw. See the Remediating fill-rate issues section of the Fill-rate, Canvases and input chapter.

If Graphic Rebuilds are using excessive CPU, as seen by a large portion of CPU time going to Canvas.SendWillRenderCanvases or Canvas::SendWillRenderCanvases, then deeper analysis is needed. Some portion of the Graphic Rebuild process is likely responsible.

In the case that a large portion of WillRenderCanvas is spent inside IndexedSet_Sort or CanvasUpdateRegistry_SortLayoutList, then time is being spent sorting the list of dirty Layout components. Consider reducing the number of Layout components on the Canvas. See Replacing layouts with RectTransforms and Splitting Canvases sections for possible remediations.

If excessive time seems to be spent in Text_OnPopulateMesh, then the culprit is simply the generation of text meshes. See the Best Fit and Disabling Canvas Renderers sections for possible remediations, and consider the advice inside Splitting Canvases if much of the text being rebuilt is not actually having its underlying string data changed.

If time is spent inside Shadow_ModifyMesh or Outline_ModifyMesh (or any other implementation of ModifyMesh), then the problem is excessive time spent calculating mesh modifiers. Consider removing these components and achieving their visual effect via static images.

If there is no particular hotspot within Canvas.SendWillRenderCanvases, or it appears to be running every frame, then the problem is likely that dynamic elements have been grouped together with static elements and are forcing the entire Canvas to rebuild too frequently. See the Splitting Canvases section.

Important reminder: Whenever any drawable UI element on a given Canvas changes, the Canvas must re-run the batch building process. This process re-analyzes every drawable UI element on the Canvas, regardless of whether it has changed or not. Note that a “change” is any change which affects a UI object’s appearance, including the sprite assigned to a sprite renderer, transform position & scale, the text contained in a text mesh, etc.

While it may seem at first glance that it is a best practice to subdivide a UI into many Subcanvases, remember that the Canvas system also does not combine batches across separate Canvases

On one Canvas, place all elements that are static and unchanging, such as backgrounds and labels. These will batch once, when the Canvas is first displayed, and then will no longer need to rebatch afterwards.

On the second Canvas, place all of the “dynamic” elements – the ones that change frequently. This will ensure that this Canvas is rebatching primarily dirty elements. If the number of dynamic elements grows very large, it may be necessary to further subdivide the dynamic elements into a set of elements that are constantly changing (e.g. progress bars, timer readouts, anything animated) and a set elements that change only occasionally.

This is actually rather difficult in practice, especially when encapsulating UI controls into prefabs. Many UIs instead elect to subdivide a Canvas by splitting out the costlier controls onto a Sub-canvas.

Given that all Raycast Targets must be tested by the Graphic Raycaster, it is a best practice to only enable the ‘Raycast Target’ setting on UI components that must receive pointer events. The smaller the list of Raycast Targets, and the shallower the hierarchy that must be traversed, the faster each Raycast test will be.

For composite UI controls that have multiple drawable UI objects that must respond to pointer events, such as a button that wishes to have its background and text both change colors, it is generally better to place a single Raycast Target at the root of the composite UI control. When that single Raycast Target receives a pointer event, it can then forward the event to each interested component within the composite control.
