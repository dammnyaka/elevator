<template>
  <div class="instruments">
    <div style="margin-bottom: 10px;">
      Queue
      <div style="display: flex; flex-wrap: wrap;">
        <div
          style="margin: 0 2px;"
          v-for="queue in localQueue"
          :key="queue.floor"
        >
          {{ queue.floor }}
        </div>
      </div>
    </div>
    <div>
      <slot name="floor" />
      <div class="btn_add">
        <button @click="removeFloor" class="btn_add-floor">-</button>
        <button @click="addFloor" class="btn_add-floor">+</button>
      </div>
    </div>
    <div>
      <slot name="elevator" />
      <div class="btn_add">
        <button @click="removeElevator" class="btn_add-elevator">
          -
        </button>
        <button @click="addElevator" class="btn_add-elevator">+</button>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    floor: {
      default: () => []
    },
    elevators: {
      default: () => []
    },
    moveElevator: {
      required: true
    },
    queue: {
      required: true
    }
  },
  data() {
    return {
      newFloorNumber: null,
      newElevatorId: null
    };
  },
  methods: {
    addFloor() {
      this.newFloorNumber = this.floor.length + 1;
      this.floor.push({ number: this.newFloorNumber, activeFloor: false });
    },
    addElevator() {
      this.newElevatorId = this.elevators.length + 1;
      this.elevators.push({
        id: this.newElevatorId,
        currentFloor: 1,
        direction: null,
        status: "idle",
        moving: null,
        targetFloor: null
      });
    },
    removeFloor() {
      this.elevators.map(i => {
        if (
          i.currentFloor > this.floor.length - 1 ||
          (i.targetFloor > this.floor.length - 1 && (i.status = "moving"))
        ) {
          clearInterval(i.moving);
          i.status = "idle";
          i.targetFloor = null;
          i.moving = null;
          i.direction = null;
          i.targetFloor = null;
          i.currentFloor = this.floor.length - 1;
          localStorage.setItem(`elevator${i.id}CurrentFloor`, i.currentFloor);
        }
      });
      this.floor.pop();
    },
    removeElevator() {
      this.elevators.pop();
    }
  }
};
</script>
<style lang="css">
.btn_add {
  position: relative;
  display: flex;
  justify-content: space-evenly;
}
.btn_add-floor {
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
