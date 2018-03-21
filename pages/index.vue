<template>
  <section class="container">
    <div>
      <logo/>
      <p :key="index" v-for="(item, index) in historicalData">
        {{cleanKey(item['key']).toLocalDate()}}
        <ul class="list-reset text-left ml-4">
          <li class="mb-1" :key="index" v-for="(data, index) in item.data">
            {{data}}
          </li>
        </ul>
      </p>
    </div>
  </section>
</template>

<script>
import Logo from "~/components/Logo.vue";
import Fire from "~/plugins/firebase.js";
const database = Fire.database();

Number.prototype.toLocalDate = function() {
  var value = new Date(this);

  value.setHours(value.getHours() - value.getTimezoneOffset() / 60);

  return value;
};

Number.prototype.toUTCDate = function() {
  var value = new Date(this);

  value.setHours(value.getHours() - value.getTimezoneOffset() / 60);

  return value;
};

export default {
  components: {
    Logo
  },
  data() {
    return {
      historicalData: []
    };
  },
  methods: {
    getWeekNumber(d) {
      d = new Date(+d);
      d.setHours(0, 0, 0, 0);
      d.setDate(d.getDate() + 4 - (d.getDay() || 7));
      var yearStart = new Date(d.getFullYear(), 0, 1);
      var weekNo = Math.ceil(((d - yearStart) / 86400000 + 1) / 7);
      return [d.getFullYear(), weekNo];
    },
    parseISOLocal(s) {
      var b = s.split(/\D/);
      return new Date(b[0], b[1] - 1, b[2]);
    },
    formattedDate(date) {
      return (
        date.getUTCDate() +
        "-" +
        (date.getUTCMonth() + 1) +
        "-" +
        date.getUTCFullYear()
      );
    },

    snapshotToArray(snapshot) {
      var returnArr = [];

      snapshot.forEach(function(childSnapshot) {
        var item = childSnapshot.val();
        item.key = childSnapshot.key;

        returnArr.push(item);
      });

      return returnArr;
    },
    getData() {
      let vm = this;
      database
        .ref("plant_daddy/historical/water_temp")
        .on("value", snapshot => {
          let snap = snapshot;
          this.historicalData = this.snapshotToArray(snapshot);
        });
    },
    cleanKey(s) {
      while (s.charAt(0) === "_") {
        s = s.substr(1);
      }
      return parseInt(s);
    },
    getDate(d) {
      return new Date(parseInt(d));
    }
  },
  created() {
    this.getData();
  },
  computed: {},
  mounted() {
    setTimeout(x => {
      this.historicalData.forEach((x, index) => {
        let s = this.cleanKey(x["key"]);
        let date = s.toLocalDate();
        console.log(date);
        // var d = new Date(parseInt(s));
        // console.log("hhhhh");
        // let test = this.getWeekNumber(d)
        var formattedDate =
          date.getUTCDate() +
          "-" +
          (date.getUTCMonth() + 1) +
          "-" +
          date.getUTCFullYear();

        console.log(this.formattedDate(date));
      });
    }, 1000);
  }
};
</script>

<style>
.container {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}
.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; /* 1 */
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}
.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}
.links {
  padding-top: 15px;
}
</style>
