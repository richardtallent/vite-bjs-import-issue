<template>
	<canvas ref="renderCanvas" id="renderCanvas" />
</template>

<script>
import { ref, onMounted, onUpdated, onUnmounted } from "vue";
import * as BABYLON from "@babylonjs/core";
import "babylonjs-procedural-textures";

export default {
	name: "App",
	setup() {
		let isInitialized = false;
		let state = ref({ refreshCount: 0 });

		const initBabylon = async function () {
			if (isInitialized) return;

			const env = {
				canvas: document.getElementById("renderCanvas"),
				engine: null,
				scene: null,
				isImmersiveMode: false,
				xrHelper: null,
			};
			env.engine = new BABYLON.Engine(env.canvas, true);
			env.scene = new BABYLON.Scene(env.engine);
			const light = new BABYLON.HemisphericLight(
				"light",
				new BABYLON.Vector3(0, 1, 0),
				env.scene
			);

			const fireMaterial = new BABYLON.StandardMaterial("matFloor");
			var fireTexture = new BABYLON.FireProceduralTexture(
				"fire",
				256,
				env.scene
			);
			fireMaterial.diffuseTexture = fireTexture;
			fireMaterial.opacityTexture = fireTexture;

			const fireball = BABYLON.MeshBuilder.CreateSphere(
				"fireball",
				{ diameter: 1 },
				env.scene
			);
			fireball.position.z = 10;
			fireball.position.y = 1;

			// Start up the non-VR mode
			const defaultCamera = new BABYLON.UniversalCamera(
				"cameraNormal",
				new BABYLON.Vector3.Zero(),
				env.scene
			);
			defaultCamera.setTarget(new BABYLON.Vector3(0, 1, 10));
			defaultCamera.attachControl(env.canvas, true);
			defaultCamera.checkCollisions = true;

			// Initialize the helper to enter/exist VR mode, if available
			try {
				env.xrHelper = await env.scene.createDefaultXRExperienceAsync({
					floorMeshes: [env.room.ground],
				});
			} catch (e) {
				console.log("No XR support detected");
			}

			env.engine.runRenderLoop(function (engine) {
				env.scene.render();
			});

			window.addEventListener("resize", function () {
				console.log("resizing");
				env.engine.resize();
			});
			isInitialized = true;
		};

		onMounted(async () => {
			initBabylon();
		});

		return { state };
	},
};
</script>
<style scoped>
#renderCanvas {
	width: 100%;
	height: 100%;
	touch-action: none;
}
</style>
