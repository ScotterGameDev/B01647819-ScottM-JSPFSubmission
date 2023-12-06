# Programming Fundamentals
## Lava Hopper (Element 5)
### Building The Level

To create Lava Hopper I have needed to use functions to create many things to have a complete end product game created by me. Such as:

- Terrain from heightmaps
- Ground
- Players character
- Platforms for the player to walk on
- Floaty rotating end cubes for the player to aim for
- Scary lava for the player to avoid

and much more!

**Terrain**

My element 5 for this project (Lava Hopper) is a game where the player must navigate a pathway floating on top of lava which, if they get to the end, they win! If they do not get to the end and fall into the lava, they of course, lose the game.

Lava hopper consists of a game scene that is initally built on top of a height map terrain and has a grey rocky material applied to it. This creates an indentation on the land that encapsulates the player so they can more easily see the play area.

Below, you can see how this terrain was created. This is the function used to create the terrain, it uses a meshbuilder and finds the terrain mesh from an online reference, inside the mesh building method. The material applied to the terrain is supplied locally so the file reference is a local one, instead of an online one.

```javascript
function createTerrain(scene: Scene)
{
  //Create large ground for valley environment
  const largeGroundMat = new StandardMaterial("largeGroundMat");
  largeGroundMat.diffuseTexture = new Texture("textures/rock.jpg");
  
  const largeGround = MeshBuilder.CreateGroundFromHeightMap("largeGround", "https://assets.babylonjs.com/environments/villageheightmap.png", {width:150, height:150, subdivisions: 20, minHeight:0, maxHeight: 10});
  largeGround.material = largeGroundMat;
  return largeGround;
}
```

**Lava**

When it came to creating the lava effect, I went through multiple iterations of getting it to look as good as I could. to begin with, in my earlier elements, I had this being one object that would rotate and would gather collisions too, but adding the player in and trying to get those collions, meant the rotation stopped. So, I decided to have two seperate objects, slightly differing in y position, so that the top one, could rotate slowly and look more lava-like, while the bottom one would not spin but handle the collisions.

So, as you can see I used pretty much the same function but added another method to one of them to secure the rotation which would only happen on the y axis and very slowly. Both are utilising the in-built funciton of creating a ground, which will create a thin, almost 2D, plane like shape, and then having a material applied afterwards, using the objects tag, to then have a rotating plane, with a lava material. gently spinning, while also gathering collisions.

```javascript
function createLava(scene: Scene)
{
  //Create Village ground
  const groundMat = new StandardMaterial("groundMat");
  groundMat.diffuseTexture = new Texture("textures/lava2.jpg");
  groundMat.diffuseTexture.hasAlpha = true;

  const ground = MeshBuilder.CreateGround("ground", {width:105, height:105});
  ground.material = groundMat;
  ground.position.y = 3.5;
  
  scene.registerAfterRender(function () {
    ground.rotate(new Vector3(0, 1, 0)/*axis*/, 0.00025/*angle*/, Space.LOCAL);
  });
  
  return ground;
}
```

```javascript
function createLavaCollider(scene: Scene)
{
  //Create Village ground
  const groundMat = new StandardMaterial("groundMat");
  groundMat.diffuseTexture = new Texture("textures/lava2.jpg");
  groundMat.diffuseTexture.hasAlpha = true;

  const ground = MeshBuilder.CreateGround("ground", {width:105, height:105});
  ground.material = groundMat;
  ground.position.y = 3;
  
  return ground;
}
```

**Rocks**

Sprites were also used to give the lava some height and make it look a little more real. This method would take the png that was brought over into the project file and do a sort of 'billboarding' effect. this was borrowed from a previous attempt at the sprite usage which has the sprites tagged as trees. This function will take the billboarded png's, ensure they always look towards the camera and use a for loop to create as many as the developer wants, in random locations set around the level, of course in set parameters (as you can see from the 'position' parameters).

```javascript
function createSpriteRocks(scene: Scene)
{
  const spriteManagerRocks = new SpriteManager("RockManager", "textures/lava_rocks.png", 2000, {width: 512, height: 1024}, scene);

  //We create trees at random positions
  for (let i = 0; i < 300; i++) {
    const tree = new Sprite("tree", spriteManagerRocks);
    tree.position.x = Math.random() * -60 + 30;
    tree.position.z = Math.random() * -60 + 30;
    tree.position.y = 3.5;
  }

  return spriteManagerRocks;
}
```

**Lighting**

The lighting for the scene was also important to help portray the lava effect and keep the game as immersive as possible. To begin with, the first function is creating a hemispheric light that will light the whole scene so the level is at least visible to begin with. Next I created the main red spotlight that would cast over the whole level, making it seem as if the red light was coming from the lava, using diffuse and specular to ensure it spreads out and affects other lights too. Thirdly, I added a miniature spotlight over the end level goal, to try and make it stand out to the player as a place they need to reach.

``` javascript
function createLight(scene: Scene) 
{
  const light = new HemisphericLight("light", new Vector3(0, 1, 0), scene);
  light.intensity = 0.5;
  return light;
}
```

```javascript
function createSpotLightMain(scene: Scene, px: number, py: number, pz: number){
  var light = new SpotLight("spotLight", new Vector3(0, 40, 0), new Vector3(0, -1, 0), Math.PI / 1, 1, scene);
  light.diffuse = new Color3(1, 0, 0);
  light.specular = new Color3(1, 0, 0);
  return light;
}
```

```javascript
function createSpotLightMini(scene: Scene, px: number, py: number, pz: number){
  var light = new SpotLight("spotLight", new Vector3(-24, 9, 19), new Vector3(0, -1, 0), Math.PI / 1, 1, scene);
  light.diffuse = new Color3(1, 0, 0);
  light.specular = new Color3(1, 0, 0);
  return light;
}
```

**Camera**

Now for the camera that the player uses to survey the level. I chose to use an arc rotate camera so the player could be in *almost* full control of the camera during gameplay. I ensured the camera was at a distance that would suit being able to see the full level and the it would be fully focused on the center of the level too. Afterwards, I added panning limits, so the camera could not go under the level, I also added other parameters for how many times the camera could turn around the level but felt it was unnecessary for the game I was creating so I commented it out in case I needed it later.

```javascript
function createArcRotateCamera(scene: Scene, player: Mesh) {
  let camAlpha = -Math.PI / 2,
    camBeta = Math.PI / 2.5,
    camDist = 10,
    camTarget = new Vector3(0, 0, 0);
  let camera = new ArcRotateCamera(
    "camera1",
    camAlpha,
    camBeta,
    camDist,
    camTarget,
    scene,
  );

  camera.lowerRadiusLimit = 50;
  camera.upperRadiusLimit = 55;
  //camera.lowerAlphaLimit = 0;
  //camera.upperAlphaLimit = Math.PI * 2;
  camera.lowerBetaLimit = 0;
  camera.upperBetaLimit = Math.PI / 2.5;

  camera.attachControl(true);
  return camera;
}
```

**Platforms**

The platforms the player will walk on were created manually but then merged into a single mesh for the player to walk over. Originally this was going to be a hand made maze but as the 3D object was having issues creating a collider, I chose to instead create a pathway manually that the player would walk on. I done this by using a function to create the platforms firstly (using a box mesh builder and scaling it out to suit the platforms size), and then another one which would take all of those platforms into account, merge them together, and then apply a collider to each so that the player could fully interact with them and walk across them to get to the finish.

```javascript
function createPlatform(scene: Scene, x: number, y: number, z: number, sx: number, sy: number, sz: number){
  let box: Mesh = MeshBuilder.CreateBox("box");
  box.position.x = x;
  box.position.y = y;
  box.position.z = z;
  box.scaling = new Vector3(sx, sy, sz);
  box.material = new StandardMaterial("blackRock", scene);

  const cubeMat = new StandardMaterial("cubeMat");
  cubeMat.diffuseTexture = new Texture("textures/black_rock.jpg");
  box.material = cubeMat;

  const boxAggregate = new PhysicsAggregate(box, PhysicsShapeType.BOX, { mass: 100 }, scene);
  return box;

}
```

```javascript
function mergeMeshes(scene: Scene, plat1: Mesh, plat2: Mesh, plat3: Mesh, plat4: Mesh, plat5: Mesh, plat6: Mesh, plat7: Mesh, plat8: Mesh, plat9: Mesh, plat10: Mesh, plat11: Mesh, plat12: Mesh)
{
  let mergedMesh = Mesh.MergeMeshes([plat1, plat2, plat3, plat4, plat5, plat6, plat7, plat8, plat9, plat10, plat11, plat12]);

  const meshAggregate1 = new PhysicsAggregate(plat1, PhysicsShapeType.BOX, { mass: 0 }, scene);
  const meshAggregate2 = new PhysicsAggregate(plat2, PhysicsShapeType.BOX, { mass: 0 }, scene);
  const meshAggregate3 = new PhysicsAggregate(plat3, PhysicsShapeType.BOX, { mass: 0 }, scene);
  const meshAggregate4 = new PhysicsAggregate(plat4, PhysicsShapeType.BOX, { mass: 0 }, scene);
  const meshAggregate5 = new PhysicsAggregate(plat5, PhysicsShapeType.BOX, { mass: 0 }, scene);
  const meshAggregate6 = new PhysicsAggregate(plat6, PhysicsShapeType.BOX, { mass: 0 }, scene);
  const meshAggregate7 = new PhysicsAggregate(plat7, PhysicsShapeType.BOX, { mass: 0 }, scene);
  const meshAggregate8 = new PhysicsAggregate(plat8, PhysicsShapeType.BOX, { mass: 0 }, scene);
  const meshAggregate9 = new PhysicsAggregate(plat9, PhysicsShapeType.BOX, { mass: 0 }, scene);
  const meshAggregate10 = new PhysicsAggregate(plat10, PhysicsShapeType.BOX, { mass: 0 }, scene);
  const meshAggregate11 = new PhysicsAggregate(plat11, PhysicsShapeType.BOX, { mass: 0 }, scene);
  const meshAggregate12 = new PhysicsAggregate(plat12, PhysicsShapeType.BOX, { mass: 0 }, scene);
  plat12.receiveShadows = true;

  return mergedMesh;
}
```

**The Player**

This section of the player programming, showcases how the player will move, while also setting booleans for animating. By checking for which key has been pressed, the player character will either move forward by using the "moveWithCollisions" method, backwards by using the same method but with *minus* speed, or rotate left and right by checking the button and then rotating in the correct direction at the set rotation speed.

```javascript
        let keydown: boolean = false;
        if (keyDownMap["w"] || keyDownMap["ArrowUp"]) {
          mesh.moveWithCollisions(mesh.forward.scaleInPlace(speed));                
          keydown = true;
        }
        if (keyDownMap["a"] || keyDownMap["ArrowLeft"]) {
          mesh.rotate(Vector3.Up(), -rotationSpeed);
          keydown = true;
        }
        if (keyDownMap["s"] || keyDownMap["ArrowDown"]) {
          mesh.moveWithCollisions(mesh.forward.scaleInPlace(-speedBackward));
          keydown = true;
        }
        if (keyDownMap["d"] || keyDownMap["ArrowRight"]) {
          mesh.rotate(Vector3.Up(), rotationSpeed);
          keydown = true;
        }
```

This is where the players animating happens. firstly, the script will set a boolean for if the object is currently animating or not. By checking if the player is moving and has already started animating, the code will only be called once, whereas if we didn't check if the player was animating, the code would continue to be called and would result in major frame rate issues. This block of code also checks (in the last section) if the player is still playing the walking animation but is not moving, in which case, play the idle animation and reset the original animation trigger boolean, so the whole thing can be re-triggered again when the player starts moving again.

```javascript
        let isPlaying: boolean = false;
        if (keydown && !isPlaying) {
          if (!animating) {
            idleAnim = scene.stopAnimation(skeleton);
            walkAnim = scene.beginWeightedAnimation(skeleton, walkRange.from, walkRange.to, 1.0, true);
            animating = true;
          }
          if (animating) {
            isPlaying = true;
          }
        } else {
          if (animating && !keydown) {
            walkAnim = scene.stopAnimation(skeleton);
            idleAnim = scene.beginWeightedAnimation(skeleton, idleRange.from, idleRange.to, 1.0, true);
            animating = false;
            isPlaying = false;
          }
        }
```

Finally (for the level), we have the collision triggers that are stored on the player, indicating both win and loss conditions. This will check if the player is interacting with the platforms and return a console message to ensure the player is in fact on the platforms and not hitting the lava. Then, the collision is checking if the player has collided with the level objective floating box, if so, reset the players position back to the beginning and reset the rotation too, create an audio method to play the *winning* sound, then play it and set the scene index to load the *game won* scene. This is then done again but for if the player has fallen off the level and hit the lava, the player will be reset, although no noise will play and the player will be sent to the *you lost* scene. Also inside the collisions is the minor adjustments of speed. this was done as when the scene was reset, the players speed would increase slightly, a quick fix was to minorly adjust this whenever the player is reset to get the game playing normally for as long as possible before needing to be refreshed and reset. (I have a feeling the movement speed is also frame-based so I didn't want to adjust the speed too much as then that could turn into problems between different PC's)

```javascript
        //collision
        if (mesh.intersectsMesh(colliderGround)) {
          console.log("COLLIDED");
        }        
        if (mesh.intersectsMesh(endBox)) {
          console.log("YOUWON");
          mesh.rotation = new Vector3(0, 0, 0);
          mesh.position.x = 18;
          mesh.position.y = 8;
          mesh.position.z = -23;

          speed -= 0.0015;
          speedBackward -= 0.0008;
          rotationSpeed -= 0.002;

          const wonSound = new Sound("wonSFX", "./audio/winner.wav", scene, null, {
            loop: false,
            autoplay: false,
          });   

          wonSound.play();
          setSceneIndex(2);
        }    
        if (mesh.intersectsMesh(colliderLava)) {
          console.log("YOULOST");
          mesh.rotation = new Vector3(0, 0, 0);
          mesh.position.x = 18;
          mesh.position.y = 8;
          mesh.position.z = -23;

          speed -= 0.0015;
          speedBackward -= 0.0008;
          rotationSpeed -= 0.002;

          setSceneIndex(3);
        }
```

### Creating The Main Menu

**Start Game Button**

This is how I created the *start game* button featured in the main menu. As you can see I used the GUI method to create a simple button. I created parameters to decide where the button should be placed in the scene, how wide and tall it should be, the button colour, how smoothed out the edges are and then the buttons background colour. Below that I created two sounds to play when the button is pressed, one to register the button click and another to play throughout the level, making sure it is set to loop. Then, when the button is pressed, I play the two sounds and change the scene.

```javascript
  function createSceneButton(scene: Scene, name: string, index: string, x: string, y: string, advtex) {
    let button = GUI.Button.CreateSimpleButton(name, index);
        button.left = x;
        button.top = y;
        button.width = "160px";
        button.height = "60px";
        button.color = "white";
        button.cornerRadius = 20;
        button.background = "red";

        const buttonClick = new Sound("ambienceSFX", "./audio/ambience.wav", scene, null, {
          loop: true,
          autoplay: false,
        });        
        const buttonClick2 = new Sound("MenuClickSFX", "./audio/click.wav", scene, null, {
          loop: false,
          autoplay: false,
        });

        button.onPointerUpObservable.add(function() {
            console.log("THE BUTTON HAS BEEN CLICKED");
            setSceneIndex(1);
            buttonClick.play();
            buttonClick2.play();
        });
        advtex.addControl(button);
        return button;
 }
```

**Main menu Text**

Using this method I created the text displayed on the menu screen. This again used a GUI method to create a text block which has its location set in the same way as the button, then setting the content, size and colour of the text too. This was then done for more lines of text, choosing to do it this way so I had easy control of the placement and parameters of each line, instead of trying to do it all in the same function.

```javascript
 function createTutorialHeader(scene: Scene, name: string, index: string, x: string, y: string, advtex)
 {
  let tutText = new GUI.TextBlock();
  tutText.left = x;
  tutText.top = y;
  tutText.text = "Welcome to Lava Hopper!";
  tutText.color = "white";
  tutText.fontSize = 42;
  advtex.addControl(tutText);
 } 
```

**Game Won and Game Lost**

Game Won and Game lost are created the exact same way as the main menu above, just with the buttons directing back to the main menu and the text boxes having slightly different text to signify if the player won or lost.

### Misc
**Skybox**

Throughout every scene and element I have used a skybox to give the GUI and game scene a background. This is done by creating a huge box that the game will take place in and ensuring that the inside of the box also has a material applied by setting backFaceCulling to true, so that the material is seen from the inside (where the game is played). This uses a cube texture which needs 6 different images, all applied to each side of the box. This is done by adding all 6 images to a folder and giving them the same naming convention with a "_*(co-ord)*" attatched the the end, allowing the function to choose where the images go with your guidance, when used alongside the coordinates mode used just below.

```javascript
function createSkybox(scene: Scene){
  const skybox = MeshBuilder.CreateBox("skyBox", {size:150}, scene);
	  const skyboxMaterial = new StandardMaterial("skyBox", scene);
	  skyboxMaterial.backFaceCulling = false;
	  skyboxMaterial.reflectionTexture = new CubeTexture("textures/space", scene);
	  skyboxMaterial.reflectionTexture.coordinatesMode = Texture.SKYBOX_MODE;
	  skyboxMaterial.diffuseColor = new Color3(0, 0, 0);
	  skyboxMaterial.specularColor = new Color3(0, 0, 0);
	  skybox.material = skyboxMaterial;
    return skybox;
}
```

**Final calling of functions**

The scripting below showcases how all of these functions listed above were called into the scenes. The first section being used to create references to things these functions will create, ensuring they are targetting the right Engine parameter. Then further down you can see the functions being called with all of the necessary parameters of those methods also being filled in for things like collisions, positions, scales, etc.

```javascript
export default function createStartScene(engine: Engine) {
  interface SceneData {
    scene: Scene;
    faceBox?: Mesh;
    light?: Light;
    terrain?: Mesh;
    lava?: Mesh;
    lavaCollider?: Mesh;
    trees?: SpriteManager;
    importMesh?: any;
    rocks?: SpriteManager;
    skybox?: Mesh;
    plat1?: Mesh;
    plat2?: Mesh;
    plat3?: Mesh;
    plat4?: Mesh;
    plat5?: Mesh;
    plat6?: Mesh;
    plat7?: Mesh;
    plat8?: Mesh;
    plat9?: Mesh;
    plat10?: Mesh;
    plat11?: Mesh;
    plat12?: Mesh;
    box?: Mesh;
    sphere?: Mesh;
    roof?: Mesh;
    house?: Mesh;
    mergedMesh?: any;
    spotLightMain?: SpotLight;
    spotLightMini?: SpotLight;
    actionManager?: any;
    camera?: Camera;
  }

  let that: SceneData = { scene: new Scene(engine) };
  that.scene.debugLayer.show();

  //initialise physics
  that.scene.enablePhysics(new Vector3(0, -9.8, 0), havokPlugin);

  that.plat1 = createPlatform(that.scene, 18, 4, -16, 3, 1, 20);
  that.plat2 = createPlatform(that.scene, 13, 4, -7.5, 8, 1, 3);
  that.plat3 = createPlatform(that.scene, 10.2, 4, -14.1, 3, 1, 10);
  that.plat4 = createPlatform(that.scene, 7, 4, -20, 9, 1, 3);
  that.plat5 = createPlatform(that.scene, 4, 4, -6, 3, 1, 26);
  that.plat6 = createPlatform(that.scene, -4.5, 4, 5.5, 14, 1, 3);
  that.plat7 = createPlatform(that.scene, -10, 4, -3, 3, 1, 15);
  that.plat8 = createPlatform(that.scene, -15.5, 4, -12, 14, 1, 3);
  that.plat9 = createPlatform(that.scene, -21, 4, 2, 3, 1, 25);
  that.plat10 = createPlatform(that.scene, -2, 4, 13, 35, 1, 3);
  that.plat11 = createPlatform(that.scene, 14, 4, 17.5, 3, 1, 6);
  that.plat12 = createPlatform(that.scene, -7.5, 4, 19, 40, 1, 3);
  that.mergedMesh = mergeMeshes(that.scene, that.plat1, that.plat2, that.plat3, that.plat4, that.plat5, that.plat6, that.plat7, that.plat8, that.plat9, that.plat10, that.plat11, that.plat12);
  that.rocks = createSpriteRocks(that.scene);
  that.terrain = createTerrain(that.scene);
  that.lava = createLava(that.scene);
  that.lavaCollider = createLavaCollider(that.scene);
  that.skybox = createSkybox(that.scene);
  that.light = createLight(that.scene);
  that.box = createEndBox(that.scene, -24, 6, 19, 0.75, 0.75, 0.75);
  that.spotLightMain = createSpotLightMain(that.scene, 0, 6, 0);
  that.spotLightMini = createSpotLightMini(that.scene, 0, 8, 0);
  that.camera = createArcRotateCamera(that.scene, that.importMesh);
  that.importMesh = importPlayerMesh(that.scene, that.mergedMesh, that.box, that.lavaCollider, 18, 8, -23, 1.5, 1.5, 1.5);
  that.actionManager = actionManager(that.scene);
  return that;
}
```