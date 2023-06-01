<template>
  <div class="wrapper">
    <div class="elev" v-for="(elevator, index) in elevators" :key="index">
      <elevator-shaft
        :floors-number="floors.length"
        :current-floor="elevator.currentFloor"
        :queue="queue"
        :elevator="elevator"
      ></elevator-shaft>
    </div>
    <div class="floors">
      <floor
        v-for="floor in floors"
        :key="floor.number"
        :number="floor.number"
        :activeFloor="floor.activeFloor"
        @call-elevator="handleCallElevator"
      ></floor>
    </div>
    <div class="instruments">
      <instruments
        :floor="floors"
        :elevators="elevators"
        :moveElevator="moveElevator"
        :queue="queue"
        ><template v-slot:floor>Floor</template
        ><template v-slot:elevator>Elevator</template></instruments
      >
    </div>
  </div>
</template>

<script>
import ElevatorShaft from "./components/ElevatorShaft.vue";
import Floor from "./components/Floor.vue";
import Instruments from "./components/Instruments.vue";

export default {
  components: {
    ElevatorShaft,
    Floor,
    Instruments
  },
  data() {
    return {
      floors: [
        { number: 1, activeFloor: false },
        { number: 2, activeFloor: false },
        { number: 3, activeFloor: false },
        { number: 4, activeFloor: false },
        { number: 5, activeFloor: false }
      ],
      elevators: [
        {
          id: 1,
          currentFloor: 1,
          direction: null,
          status: "idle",
          moving: null,
          targetFloor: null
        },
        {
          id: 2,
          currentFloor: 1,
          direction: null,
          status: "idle",
          moving: null,
          targetFloor: null
        }
      ],
      queue: []
    };
  },
  created() {
    this.elevators.map(elevator => {
      const loadElevatorFloor = parseInt(
        localStorage.getItem(`elevator${elevator.id}CurrentFloor`),
        10
      );

      if (!isNaN(loadElevatorFloor)) {
        elevator.currentFloor = loadElevatorFloor;
      }
    });
  },
  methods: {
    handleCallElevator(floorNumber) {
      const elevatorOnFloor = this.elevators.find(
        elevator =>
          elevator.currentFloor === floorNumber ||
          elevator.targetFloor === floorNumber
      );
      const floorInQueue = this.queue.some(call => call.floor === floorNumber);

      if (elevatorOnFloor || floorInQueue) {
        return;
      } else {
        this.floors[floorNumber - 1].activeFloor = true;
        const freeElevator = this.elevators.find(
          elevator => elevator.status === "idle"
        );

        if (freeElevator) {
          freeElevator.targetFloor = floorNumber;
          freeElevator.status = "moving";
          freeElevator.direction =
            floorNumber > freeElevator.currentFloor ? "up" : "down";
          this.moveElevator(this.elevators.indexOf(freeElevator));
        } else {
          this.queue.push({ floor: floorNumber });
        }
      }
    },
    moveElevator(index) {
      const elevator = this.elevators[index];

      if (this.queue.length === 0 && elevator.status === "idle") {
        elevator.status = "idle";
        return;
      }

      if (this.queue.length > 0) {
        const call = this.queue[0];
        const floorNumber = call.floor;
        const direction = floorNumber > elevator.currentFloor ? "up" : "down";
        elevator.targetFloor = floorNumber;
        elevator.status = "moving";
        elevator.direction = direction;
        this.queue.shift();
      }

      elevator.moving = setInterval(() => {
        if (elevator.currentFloor === elevator.targetFloor) {
          clearInterval(elevator.moving);
          elevator.status = "resting";
          elevator.direction = null;
          this.floors[elevator.targetFloor - 1].activeFloor = false;
          elevator.targetFloor = null;

          setTimeout(() => {
            return this.queue.length > 0
              ? this.moveElevator(index)
              : (elevator.status = "idle");
          }, 3000);
          return;
        }

        elevator.currentFloor += elevator.direction === "up" ? 1 : -1;
        localStorage.setItem(
          `elevator${elevator.id}CurrentFloor`,
          elevator.currentFloor
        );
      }, 1000);
    }
  }
};
</script>

<style lang="css">
.wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
.elev {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-right: 20px;
}
.floors {
  display: flex;
  flex-direction: column-reverse;
}
.instruments {
  position: relative;
  width: 100px;
  margin-left: 10px;
  background-color: #eee;
  text-align: center;
}
</style>
