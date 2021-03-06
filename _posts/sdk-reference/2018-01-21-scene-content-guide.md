---
date: 2018-06-12
title: Scene content guide
description: Learn how to import 3D models to use in a scene
categories:
  - sdk-reference
type: Document
set: sdk-reference
set_order: 2
tag: introduction
---

Three dimensional scenes in Decentraland are based on the [Entity-Component](https://en.wikipedia.org/wiki/Entity%E2%80%93component%E2%80%93system) model, where everything in a scene is an *entity*, and each entity can include *components* that shape its characteristics and functionality. 

Entities can be nested inside others to create a tree structure. In fact, all scenes must output a tree of nested entities. `.xml` scenes explicitly define this tree, `.xlt` scenes define typescript code that builds and updates this tree.

This document covers how to achieve common objectives by using different types of entities and components in a scene's tree.

## Create simple geometric shapes

Several basic shapes, often called *primitives*, can be added to a scene as predefined entity types. These already have certain components defined (like their shape) and let you set others (like their rotation and color).

The following types of entities are available:

* `<Box />`
* `<Sphere />`
* `<Plane />`
* `<Cylinder />`
* `<Cone />`

Any of these can be added to your scene, they can all include basic components like position, scale or color.

```tsx
<box position={vector} color="#ff00aa" scale={2} />
```

See [Entity interfaces]({{ site.baseurl }}{% post_url /sdk-reference/2018-06-21-entity-interfaces %}) for more details on these types of entities.

> Tip: When editing the code via a source code editor (like Visual Studio Code), you can see the list of components supported by a type of entity. Typically, this is done by placing the cursor in the entity and typing *Ctrl + Space bar*.

## Entity positioning

All entities can have a position, a rotation and a scale. These can be easily set as components, as shown below:

{% raw %}
```tsx
<box
    position={{ x: 5, y: 3, z: 5 }}
    rotation={{ x: 180, y: 90, z: 0 }}
    scale={0.5}
  />
```
{% endraw %}

* `position` is a *3D vector*, it sets the position on all three axes. 
* `rotation` is a *3D vector* too, but where each component represents the rotation in that axis.
* `scale` can either be a *number* or a *3D vector*, in case you want to scale the axis in different proportions. 

When an entity is nested inside another, the child entities inherit components from the parents. This means that if a parent entity is positioned, scaled or rotated, its children are also affected. The position, rotation and scale values of children entities don't override those of the parents, instead these are compounded.

You can include an invisible base entity to wrap a set of other entities and define their positioning as a group.

{% raw %}
```tsx
  <entity
      position={{ x: 0, y: 0, z: 1 }}
      rotation={{ x: 45, y: 0, z: 0 }}
  >
    <box position={{ x: 10, y: 0, z: 0 }} scale={2} />
    <box position={{ x: 10, y: 10, z: 0 }} scale={1} />
    <box position={{ x: 0, y: 10, z: 0 }} scale={2} />
  </entity>
```
{% endraw %}

You can also set a position, rotation and scale for the entire <scene/> entity and affect everything in the scene.




### Transitions

In dynamic scenes, you can configure an entity to affect the way in which it moves. By default, all changes to an entity are rendered as a sudden shift from one state to another. By adding a transition component, you can make the change be gradual and more natural.

The example below shows a box entity that is configured to rotate smoothly. 

{% raw %}
```tsx
 <box 
    rotation={currentRotation}
    transition={{ rorotation: { duration: 1000, timing: "ease-in" }}}
  />
```
{% endraw %}

> Note: The transition component doesn't make the box rotate, it just sets the way it rotates whenever the value of the entity's rotation changes, usually as the result of an event.

The transition component can be added to affect the following properties of an entity:

* position
* rotation
* scale
* color

Note that the transition for each of these properties is configured separately.

{% raw %}
```tsx
 <box 
    rotation={currentRotation}
    color={currentColor}
    scale={currentScale}
    transition={ 
        { rotation: { duration: 1000, timing: "ease-in" }}
        { color: { duration: 3000, timing: "exponential-in" }}
        { scale: { duration: 300, timing: "bounce-in" }}
        }
  />
```
{% endraw %}

The transition component allows you to set:

* A delay: milliseconds to wait before the change begins occuring.
* A duration: milliseconds from when the change begins to when it ends.
* Timing: select a function to shape the transition. For example, the transition could be `linear`, `ease-in`, `ease-out`, `exponential-in` or `bounce-in`, among other options.

In the example below, a transition is applied to the rotation of an invisible entity that wraps a box. As the box is off-center from the parent entity, the box pivots like an opening door.

{% raw %}
```tsx

<entity 
    rotation={currentRotation}  
    transition={{ rotation: { duration: 1000, timing: "ease-in" }}}>
        <box 
          id="door" 
          scale={{ x: 1, y: 2, z: 0.05 }} 
          position={{ x: 0.5, y: 1, z: 0 }} 
        />
</entity>
```
{% endraw %}

## Color

Color is set in hexadecimal values. To set an entity's color, simply set its `color` component to the corresponding hexadecimal value.

{% raw %}
```tsx
  <sphere 
    position={{ x: 0.5, y: 1, z: 0 }}   
    color="#EF2D5E"
  />
```
{% endraw %}

> Tip: There are many online color-pickers you can use to find a specific color graphically. To name one, you can try the color picker on [W 3 Schools](https://www.w3schools.com/colors/colors_picker.asp).


## Materials

Materials are defined as separate entities in a scene, this prevents material definitions from being duplicated, keeping the scene's code lighter.

Materials can be applied to primitive entities and to planes, simply by setting the `material` component.

{% raw %}
```tsx
  <material 
    id="reusable_material" 
    albedoTexture="materials/wood.png" 
    roughness="0.5" 
  />
  <sphere
    material="#reusable_material" 
  />
```
{% endraw %}

Materials are also implicitly imported into a scene when you import a glTF model that includes embedded materials. When that's the case, the scene doesn't need a `<material/>` entity declared.


## Import 3D models
 
For more complex shapes, you can build a 3D model in an external tool like Blender and then import them in glTF format.  [glTF](https://www.khronos.org/gltf) (GL Transmission Format) is an open project by Khronos providing a common, extensible format for 3D assets that is both efficient and highly interoperable with modern web technologies.

> Note: When using Blender, you need an add-on to export glTF files. For models that don't include animations we recommend installing the add-on by [Kronos group](https://github.com/KhronosGroup/glTF-Blender-Exporter). To export glTFs that include animations, you should instead install the add-on by [Kupoman](https://github.com/Kupoman/blendergltf).


To add an external model into a scene, add a `<gltf-model>` element and set its `src` component to the path of the glTF file containing the model.

> Tip: We recommend keeping your models separate in a `/models` folder inside your scene.


{% raw %}
```tsx
<gltf-model
    position={{ x: 5, y: 3, z: 5 }}
    scale={0.5}
    src="models/myModel.gltf"
  />
```
{% endraw %}

glTF models can have either a `.gltf` or a `.glb` extension. glTF files are human-readable, you can open one in a text editor and read it like a JSON file. This is useful, for example, to verify that animations are properly attached and to check for their names. glb files are binary, so they're not readable but they are considerably smaller in size, which is good for the scene's performance. 

> Tip: We recommend using `.gltf` while you're working on a scene, but then switching to `.glb` when uploading it.


glTF models can also include their own textures and animations. Keep in mind that all models, their shaders and their textures must be within the parameters of the [scene limitations]({{ site.baseurl }}{% post_url /documentation/building-scenes/2018-01-06-scene-limitations %}).


> Note: obj models are also supported as a legacy feature, but will likely not be supported for much longer. To add one, use an `<obj-model>` entity. 

### Animations

> Note: Keep in mind that all models and their textures must be within the parameters of the [scene limitations]({{ site.baseurl }}{% post_url /sdk-reference/2018-01-06-scene-limitations %}).

Files with .gltf extensions can be opened with a text editor to view their contents. There you can find the list of animations included in the model and how they're named.

In a dynamic scene, you reference an animation by its armature name, an underscore and its animation name. For example `myArmature_animation1`. You activate an animation by setting its `playing` property to `true`.

The example below imports a model that includes animations and configures them:

{% raw %}
```tsx
<gltf-model
    position={{ x: 5, y: 3, z: 5 }}
    scale={0.5}
    src="models/shark_anim.gltf"
    skeletalAnimation={[
      { clip: 'shark_skeleton_bite', playing: false },
      { clip: 'shark_skeleton_swim', weight: 0.2, playing: true }
    ]}
  />
```
{% endraw %}

In this example, the armature is named `shark_skeleton` and the two animations contained in it are named `bite` and `swim`.

An animation can be set to loop continuously by setting its `loop` property. If `loop:false` then the animation will be called only once when activated.

The `weight` property allows a single model to carry out multiple animations at once, calculating a weighted average of all the movements involved in the animation. The value of `weight` determines how much importance the given animation will be given.

### Free libraries for 3D models

Instead of building your own 3d models, you can also download them from several free or paid libraries.

To get you started, below is a list of libraries that have free or relatively inexpensive content:

* [Google Poly](https://poly.google.com)
* [SketchFab](https://sketchfab.com/)
* [Clara.io](https://clara.io/)
* [Archive3D](https://archive3d.net/)
* [SketchUp 3D Warehouse](https://3dwarehouse.sketchup.com/)
* [Thingiverse](https://www.thingiverse.com/) (3D models made primarily for 3D printing, but adaptable to Virtual Worlds)
* [ShareCG](https://www.sharecg.com/)

> Note: Pay attention to the licence restrictions that the content you download has.

Note that most of the models that you can download from these sites won't be in glTF. If that's the case, you must convert them to glTF before you can use them in a scene. We recommend importing them into Blender and exporting them with one of the available glTF export add-ons.


### Why we use glTF?

Compared to the older *OBJ format*, which supports only vertices, normals, texture coordinates, and basic materials,
glTF provides a more powerful set of features. In addition to all of the features we just named, glTF also offers:

- Hierarchical objects
- Scene information (light sources, cameras)
- Skeletal structure and animation
- More robust materials and shaders

OBJ can currently be used for simple models that have no animations, but we will probably stop supporting it in the future.

Compared to *COLLADA*, the supported features are very similar. However, because glTF focuses on providing a
"transmission format" rather than an editor format, it is more interoperable with web technologies.

Consider this analogy: the .PSD (Adobe Photoshop) format is helpful for editing 2D images, but images must then be converted to .JPG for use
on the web. In the same way, COLLADA may be used to edit a 3D asset, but glTF is a simpler way of transmitting it while rendering the same result.

## Sound

You can add sound to your scene by including a sound component in any entity.


{% raw %}
```tsx
  <sphere 
    position={{ x: 5, y: 3, z: 5 }}
    sound={{
      src: "sounds/carnivalrides.ogg", 
      loop: true, 
      playing: true,
      volume: 0.5
      }}
  />
```

{% endraw %}

The `src` property points to the location of the sound file.

> Tip: We recommend keeping your sound files separate in a `/sounds` folder inside your scene.

Supported sound formats vary depending on the browser, but it's safe to use `.mp3`, `.accc` and  `.ogg`. `.wav` files are also supported but not generally recommended as they are significantly larger.

Each entity can only play a single sound file. This limitation can easily be overcome by including multiple invisible entities, each with their own sound file.

The `distanceModel` property of the sound component conditions how the user's distance to the sound's source affects its volume. The model can be `linear`, `exponential` or `inverse`. When using the linear or exponential model, you can also set the `rolloffFactor` property to set the steepness of the curve. 

<!---

Setting loop to false stops the audio, it doesn't pause it. So when setting loop to true the audio will start from the beginning.

Setting playing to false pauses??????


### How to use Blender with the SDK


how to add collider meshes into GLTF models

-->

## Entity collision

Entities that have collision disabled can be walked through by a user`s avatar, entities that do have collisions enabled occupy space and block a user's path.

{% raw %}
```tsx
<box 
  position={{ x: 10, y: 0, z: 0 }} 
  scale={2} 
  ignoreCollisions={false}
/>

```
{% endraw %}

The example above defines a box entity that can't be walked through.

All entities have collisions disabled by default. Depending on the type of entity, collisions are enabled as follows:

* For most entities, including *primitives* (boxes, spheres, etc), planes and base entities, you enable collisions by setting the `ignoreCollisions` component to `false`. 
* To enable collisions in *glTF models*, you can either:

  *   Edit them in an external tool like Blender to include a *collission mesh*.
  *   Overlay an invisible entity with the `ignoreCollisions` component set to `true`.

A *collision mesh* is a set of planes or geometric shapes that define which parts of the model are collided with. This allows for much greater control and is a lot less demanding on the system, as the collision mesh is usually a lot simpler (with less vertices) than the original model.

Collision settings currently don't affect how other entities interact with each other, entities can always overlap. Collision settings only affect how the entity interacts with the avatar.

Decentraland currently doesn't have a physics engine, so if you want entities to fall, crash or bounce, you must code this behavior into the scene.






## Migrating XML to type script

If you have a static XML scene and want to add dynamic capabilities to it, you must migrate it to TSX format. This implies making some minor changes to the entity syntax.

### Data types

> **TL;DR**  
> in XML: `position="10 10 10"`  
> in TSX: `position={ { x:10, y: 10, z: 10 } }`

There are subtle differences between the `text/xml` representation and the TSX representation of a scene. Our approach is TSX-first, and the XML representation of the scene is only a compatibility view. Because of this, attributes in TSX must be objects, not
plain text.


```xml
<scene>
  <box position="10 10 10" />
</scene>
```

The static scene above becomes the following dynamic schen when migrating it to TSX:

{% raw %}
```tsx
class Scene extends ScriptableScene {
  async render() {
    return (
      <scene>
        <box position={{ x: 10, y: 10, z: 10 }} />
      </scene>
    );
  }
}
```
{% endraw %}

#### Attribute naming

> **TL;DR**  
> in XML: `albedo-color="#ffeeaa"` (kebab-case)  
> in TSX: `albedoColor="#ffeeaa"` (camelCase)

HTML and XHTML are case insensitive for attributes, this generates conflicts with the implementation of certain attributes like `albedoColor`. Because reading `albedocolor` was confusing, and having hardcoded keys with hyphens in the code was so dirty, we decided to follow the React convention of having every property camel cased in code and hyphenated in the HTML/XML representation. 

{% raw %}
```xml
<scene>
  <!-- XML -->
  <material id="test" albedo-color="#ffeeaa" />
</scene>
```
{% endraw %}

The static scene above becomes the following dynamic schen when migrating it to TSX:


{% raw %}
```tsx
<!-- TSX -->
class Scene extends ScriptableScene {
  async render() {
    return (
      <scene>
        <material id="test" albedoColor="#ffeeaa" />
      </scene>
    );
  }
}
```
{% endraw %}
