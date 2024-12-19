<script setup>
const state = reactive({
  floors: Array.from({ length: 30 }, (_, i) => i + 1),
  elevators: [
    { position: 1, queue: [], status: 'idle', id: 1 },
    { position: 1, queue: [], status: 'idle', id: 2 },
    { position: 1, queue: [], status: 'idle', id: 3 },
    { position: 1, queue: [], status: 'idle', id: 4 },
  ],
  // floorHeight: 50,
});

const maxFloor = computed(() => state.floors.length);

function callElevator(floor, direction) {
  const elevator = findBestElevator(floor, direction);
  if (elevator) {
    elevator.queue.push(floor);
    elevator.queue = [...new Set(elevator.queue)].sort((a, b) => a - b);
    if (elevator.status === 'idle')
      moveElevator(elevator);
  }
}

function addFloorToQueue(elevator, floor) {
  elevator.queue.push(floor);
  elevator.queue = [...new Set(elevator.queue)].sort((a, b) => a - b);
  if (elevator.status === 'idle')
    moveElevator(elevator);
}

function findBestElevator(floor) {
  const availableElevators = state.elevators.map((elevator, index) => ({
    ...elevator,
    index,
    distance: Math.abs(elevator.position - floor),
  }));
  console.log('🚀 ~ availableElevators ~ availableElevators:', availableElevators);

  // Find the closest idle or moving in the same direction elevator
  availableElevators.sort((a, b) => a.distance - b.distance);
  return state.elevators[availableElevators[0].index];
}

async function moveElevator(elevator) {
  if (!elevator.queue.length) {
    elevator.status = 'idle';
    return;
  }

  elevator.status = 'moving';
  const targetFloor = elevator.queue.shift();
  const step = targetFloor > elevator.position ? 1 : -1;

  while (elevator.position !== targetFloor) {
    elevator.position += step;
    await new Promise(resolve => setTimeout(resolve, 500)); // 500ms per floor
  }

  // Simulate door opening
  elevator.status = 'opening';
  await new Promise(resolve => setTimeout(resolve, 1000)); // Door opening delay

  // Simulate door closing
  elevator.status = 'closing';
  await new Promise(resolve => setTimeout(resolve, 1000)); // Door closing delay

  // Simulate movement delay
  // const delay = Math.abs(targetFloor - elevator.position) * 500; // 500ms per floor
  // await new Promise(resolve => setTimeout(resolve, delay));
  // elevator.position = targetFloor;
  moveElevator(elevator);
}
</script>

<template>
  <div class="grid sm:flex p-5 gap-5">
    <div>
      <div>Request Floor</div>
      <div class="sm:w-[200px] h-[300px] sm:h-[calc(100vh-64px)] overflow-auto">
        <div v-for="floor in state.floors" :key="floor" class="flex items-center justify-between border border-solid mt-2 p-2 rounded">
          {{ floor }}
          <div class="flex gap-2">
            <div class="cursor-pointer" :class="{ 'pointer-events-none opacity-30': floor === maxFloor }" @click="callElevator(floor, 'up')">
              <IconTablerCircleArrowUp />
            </div>
            <div class="cursor-pointer" :class="{ 'pointer-events-none opacity-30': floor === 1 }" @click="callElevator(floor, 'down')">
              <IconTablerCircleArrowDown />
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="flex-1">
      <div>Elevator</div>
      <div class="grid sm:flex grid-cols-2 gap-5 mt-2">
        <div v-for="elevator in state.elevators" :key="elevator.id" class="flex-1 border border-solid p-2 rounded">
          <div class="mb-2 text-center">
            Elevator {{ elevator.id }}
          </div>
          <div class="grid grid-cols-5 gap-2">
            <div
              v-for="floor in state.floors" :key="floor.id"
              class="bg-gray-300 grid place-items-center h-10 rounded cursor-pointer"
              :class="{ 'bg-gray-800 text-white': elevator.position === floor }"
              @click="addFloorToQueue(elevator, floor)"
            >
              {{ floor }}
            </div>
          </div>
          <!-- <template v-else>
            <div v-for="floor in state.floors" :key="floor">
              <div class="flex justify-between">
                Floor - {{ floor }}
                <div v-if="elevator.position === floor">
                  <div v-if="elevator.status === 'opening' || elevator.status === 'closing'">
                  <div v-if="elevator.status === 'opening'" class="door-status">
                    Opening...
                  </div>
                  <div v-else-if="elevator.status === 'closing'" class="door-status">
                    Closing...
                  </div>
                </div>
                  Elevator {{ elevator.id }}
                </div>
              </div>
              <hr>
            </div>
          </template> -->
        </div>
      </div>
    </div>
  </div>
</template>
