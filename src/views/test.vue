<script>
export default {
  data() {
    return {
      floors: Array.from({ length: 30 }, (_, i) => i + 1),
      elevators: [
        { position: 1, queue: [], status: 'idle' },
        { position: 1, queue: [], status: 'idle' },
        { position: 1, queue: [], status: 'idle' },
        { position: 1, queue: [], status: 'idle' },
      ],
      floorHeight: 50, // Height in pixels for visualization
    };
  },
  computed: {
    maxFloor() {
      return this.floors.length;
    },
  },
  methods: {
    callElevator(floor, direction) {
      const elevator = this.findBestElevator(floor, direction);
      if (elevator) {
        elevator.queue.push(floor);
        elevator.queue = [...new Set(elevator.queue)].sort((a, b) => a - b);
        if (elevator.status === 'idle')
          this.moveElevator(elevator);
      }
    },
    addFloorToQueue(elevator, floor) {
      elevator.queue.push(floor);
      elevator.queue = [...new Set(elevator.queue)].sort((a, b) => a - b);
      if (elevator.status === 'idle')
        this.moveElevator(elevator);
    },
    findBestElevator(floor, direction) {
      const availableElevators = this.elevators.map((elevator, index) => ({
        ...elevator,
        index,
        distance: Math.abs(elevator.position - floor),
      }));

      // Find the closest idle or moving in the same direction elevator
      availableElevators.sort((a, b) => a.distance - b.distance);
      return this.elevators[availableElevators[0].index];
    },
    async moveElevator(elevator) {
      if (!elevator.queue.length) {
        elevator.status = 'idle';
        return;
      }

      elevator.status = 'moving';
      const targetFloor = elevator.queue.shift();
      const step = targetFloor > elevator.position ? 1 : -1;

      while (elevator.position !== targetFloor) {
        elevator.position += step;

        // Simulate movement delay for each floor
        await new Promise(resolve => setTimeout(resolve, 500)); // 500ms per floor
      }

      // Simulate door opening
      elevator.status = 'opening';
      await new Promise(resolve => setTimeout(resolve, 1000)); // Door opening delay

      // Simulate door closing
      elevator.status = 'closing';
      await new Promise(resolve => setTimeout(resolve, 1000)); // Door closing delay

      this.moveElevator(elevator); // Continue with the next request in the queue
    },
    getElevatorPosition(floor) {
      return -((floor - 1) * this.floorHeight);
    },
  },
};
</script>

<template>
  <div class="elevator-system">
    <div v-for="floor in floors" :key="floor" class="floor">
      <div class="floor-controls">
        <button :disabled="floor === maxFloor" @click="callElevator(floor, 'up')">
          Up
        </button>
        <button :disabled="floor === 1" @click="callElevator(floor, 'down')">
          Down
        </button>
      </div>
      <div class="floor-label">
        Floor {{ floor }}
      </div>
    </div>

    <div class="elevators">
      <div v-for="(elevator, index) in elevators" :key="index" class="elevator">
        <div
          class="elevator-car"
          :style="{ transform: `translateY(${getElevatorPosition(elevator.position)}px)` }"
        >
          Elevator {{ index + 1 }}
          <div v-if="elevator.status === 'opening'" class="door-status">
            Opening...
          </div>
          <div v-else-if="elevator.status === 'closing'" class="door-status">
            Closing...
          </div>
        </div>

        <!-- Elevator floor controls -->
        <div class="elevator-controls">
          <button
            v-for="floor in floors"
            :key="floor"
            :disabled="floor === elevator.position"
            @click="addFloorToQueue(elevator, floor)"
          >
            {{ floor }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.elevator-system {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  padding: 20px;
}
.floor {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.floor-controls {
  display: flex;
  flex-direction: row;
  gap: 10px;
  margin-bottom: 5px;
}
.elevators {
  display: flex;
  flex-direction: row;
  gap: 20px;
}
.elevator {
  position: relative;
  width: 120px;
  height: 1500px; /* 30 floors * 50px each */
  border: 1px solid #ccc;
  overflow: hidden;
}
.elevator-car {
  position: absolute;
  width: 80px;
  height: 50px;
  background-color: #007bff;
  color: white;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 5px;
  transition: transform 0.5s linear;
}
.elevator-controls {
  margin-top: 10px;
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
  justify-content: center;
}
.elevator-controls button {
  width: 30px;
  height: 30px;
  font-size: 0.8em;
  text-align: center;
}
.door-status {
  font-size: 0.8em;
  margin-top: 5px;
  color: #fff;
  font-weight: bold;
}
</style>
