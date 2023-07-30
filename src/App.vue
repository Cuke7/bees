<template>
  <div class="parent1 h-screen justify-around items-center flex">
    <div class="flex space-y-2 flex-col w-1/2">
      <h1 class="text-white text-9xl font-mono font-bold">Bees 🐝</h1>
      <p class="text-white text-xl font-mono">
        A bee simulation using
        <a href="https://pixijs.com/" target="_blank"
          ><span class="underline">PixiJS</span></a
        >.
      </p>
    </div>
  </div>
  <div class="h-screen bg-slate-700 flex justify-center items-center">
    <div ref="parent" class="rounded-lg w-2/3 h-2/3"></div>
  </div>
</template>

<script lang="ts" setup>
import * as PIXI from "pixi.js";
import { ref, onMounted } from "vue";

const parent = ref<HTMLDivElement | null>(null);

onMounted(() => {
  if (parent.value) {
    let size = {
      width: parent.value.getBoundingClientRect().width,
      height: parent.value.getBoundingClientRect().height,
    };
    let app = new PIXI.Application(size);
    parent.value.appendChild(app.view);

    const stage = new PIXI.Container();
    app.stage.addChild(stage);

    const radius = 5;
    const baseSpeed = 2; // Base speed for movement
    const maxSpeedVariation = 1; // Maximum variation in speed
    const maxAngleVariation = Math.PI / 4; // Maximum variation in angle (in radians)

    // Function to generate random values between -max and max
    function getRandomInRange(max) {
      return (Math.random() - 0.5) * 2 * max;
    }

    // Create flower objects
    const flowers = [];
    const numFlowers = 10; // Adjust the number of flowers as needed
    const maxHitCount = 5; // Maximum number of times a flower can be hit before disappearing

    for (let i = 0; i < numFlowers; i++) {
      const flower = new PIXI.Graphics();
      const flowerX = Math.random() * app.renderer.width;
      const flowerY = Math.random() * app.renderer.height;

      flower.beginFill(0xff0000); // Red color for flowers
      flower.drawCircle(0, 0, 8); // Draw the flower centered at (0, 0)
      flower.endFill();
      flower.position.set(flowerX, flowerY);

      stage.addChild(flower);
      flowers.push({ graphics: flower, hitCount: 0 });
    }

    // Create the hive object
    const hive = new PIXI.Graphics();
    const hiveX = app.renderer.width / 2; // Place the hive in the center of the screen
    const hiveY = app.renderer.height / 2;
    hive.beginFill(0x00ff00); // Green color for the hive
    hive.drawCircle(0, 0, 12); // Draw the hive centered at (0, 0)
    hive.endFill();
    hive.position.set(hiveX, hiveY);
    stage.addChild(hive);

    // Function to create a new bee
    function createBee() {
      const graphics = new PIXI.Graphics();
      const x = Math.random() * app.renderer.width;
      const y = Math.random() * app.renderer.height;

      graphics.beginFill(0xffff00); // Yellow color for bees
      graphics.drawCircle(0, 0, radius); // Draw the circle centered at (0, 0)
      graphics.endFill();
      graphics.position.set(x, y);

      stage.addChild(graphics);

      return {
        graphics,
        x,
        y,
        seekingFlower: true,
        lastFlowerTarget: null,
      };
    }

    // Create an array to store the bees
    const bees = [];
    const numBees = 5; // Adjust the number of bees as needed

    for (let i = 0; i < numBees; i++) {
      bees.push(createBee());
    }

    app.ticker.add(() => {
      for (const bee of bees) {
        // Find the nearest flower or hive based on current target
        let nearestDistance = Number.MAX_VALUE;
        let nearestTarget;

        if (bee.seekingFlower) {
          // If the bee already made contact with a flower
          if (bee.lastFlowerTarget) {
            const flower = flowerData.graphics;

            const distance = Math.sqrt(
              (bee.x - flower.x) ** 2 + (bee.y - flower.y) ** 2
            );

            // if (distance < nearestDistance) {
            nearestDistance = distance;
            nearestTarget = flower;
            // }
          } else {
            // Find the nearest flower
            for (const flowerData of flowers) {
              const flower = flowerData.graphics;

              const distance = Math.sqrt(
                (bee.x - flower.x) ** 2 + (bee.y - flower.y) ** 2
              );

              if (distance < nearestDistance) {
                nearestDistance = distance;
                nearestTarget = flower;
              }
            }
          }
        } else {
          // Find the hive
          nearestTarget = hive;
          nearestDistance = Math.sqrt(
            (bee.x - hive.x) ** 2 + (bee.y - hive.y) ** 2
          );
        }

        // If there is a nearest target, update the bee's movement direction
        if (nearestTarget) {
          const angle = Math.atan2(
            nearestTarget.y - bee.y,
            nearestTarget.x - bee.x
          );
          const speedVariation = getRandomInRange(maxSpeedVariation);
          const angleVariation = getRandomInRange(maxAngleVariation);

          bee.x +=
            (baseSpeed + speedVariation) * Math.cos(angle + angleVariation);
          bee.y +=
            (baseSpeed + speedVariation) * Math.sin(angle + angleVariation);

          // Keep the point within the screen bounds
          if (bee.x < -radius) bee.x = app.renderer.width + radius;
          if (bee.x > app.renderer.width + radius) bee.x = -radius;
          if (bee.y < -radius) bee.y = app.renderer.height + radius;
          if (bee.y > app.renderer.height + radius) bee.y = -radius;

          // Update the position of the circle
          bee.graphics.position.set(bee.x, bee.y);

          // Check if the bee has made contact with the current target
          if (nearestDistance <= radius + 12) {
            if (nearestTarget === hive) {
              bee.seekingFlower = true;
            } else {
              // Handle flower hit count
              const flowerData = flowers.find(
                (f) => f.graphics === nearestTarget
              );
              flowerData.hitCount++;

              // Update the bee flowerTarget
              // if (flowerData) bee.lastFlowerTarget = flowerData;

              // Check if the flower should disappear
              if (flowerData.hitCount >= maxHitCount) {
                stage.removeChild(nearestTarget);
                flowers.splice(flowers.indexOf(flowerData), 1);
              }

              // Reset seekingFlower to false so the bee seeks the hive again
              bee.seekingFlower = false;
            }
          }
        }
      }
    });
  }
});
</script>

<style scoped>
.parent1 {
  background-image: linear-gradient(
    to right bottom,
    #051937,
    #00476d,
    #00787b,
    #00a655,
    #b5c600
  );
}
</style>
