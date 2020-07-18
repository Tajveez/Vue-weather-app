<template>
  <div id="app">
    <main>
      <div class="welcome-heading">Welcome, User</div>
      <div class="search-box">
        <input
          type="text"
          class="search-bar"
          placeholder="What's on your Mind?"
          v-model="query"
          @keypress="getWeather"
        />
      </div>
      <div v-if="errorStr">Please, Turn on your location</div>
      <div class="weather-wrap" v-else>
        <div class="location-box">
          <div class="location">
            <i class="fa fa-map-marker" aria-hidden="true"></i>
            {{weather.name}}, {{weather.sys.country}}
          </div>
          <div class="date">{{getCurrentDate()}}</div>
          <!-- <div class="date">Monday, 20 July 2020</div> -->
        </div>
        <div class="weather-box">
          <div class="temp-box">
            <div class="temp">
              {{weather.main.temp}}° C
              <hr />
            </div>
            <div class="temp-desc">
              <i class="fas fa-sun"></i>
              <!-- <i class="fas fa-sun"></i> -->
              {{getTime(weather.sys.sunrise)}} |
              <i class="fas fa-moon"></i>
              {{getTime(weather.sys.sunset)}}
            </div>
          </div>
          <div class="weather">
            <!-- <img id="wicon" :src="weatherIcon" alt="Weather icon" /> -->
            {{weather.weather[0].main}}
          </div>
          <div class="weather-desc">{{weather.weather[0].description}}</div>
        </div>
      </div>
      <!-- <a class="change-background" @click="changeBackground">
        <i class="fas fa-reload"></i>
      </a>-->
    </main>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      api_key: "6b53442b5b896092cedb03a4b53e38a7",
      url_base: "https://api.openweathermap.org/data/2.5/",
      query: "",
      weather: {},
      monthName: [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December"
      ],
      dayName: [
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
        "Sunday"
      ],
      location: null,
      gettingLocation: false,
      errorStr: null
    };
  },
  components: {},
  computed: {
    weatherIcon() {
      let url =
        "http://openweathermap.org/img/w/" +
        this.weather.weather[0].icon +
        ".png";
      return url;
    }
  },
  methods: {
    changeBackground() {
      let url =
        "https://source.unsplash.com/random/1600x900?nature,water,art,space,wild,happy,city,food,history,animal,car";
      let app = document.getElementById("app");
      app.style.backgroundImage = `url('${url}')`;
    },
    getCurrentDate() {
      let today = new Date(this.weather.dt * 1000);

      let date =
        this.dayName[today.getDay() - 1] +
        ", " +
        today.getDate() +
        " " +
        this.monthName[today.getMonth()] +
        " " +
        today.getFullYear();

      return date;
    },
    getWeather(e) {
      if (e.key == "Enter") {
        fetch(
          `${this.url_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}`
        )
          .then(res => {
            return res.json();
          })
          .then(this.setWeather);
      }
    },
    setWeather(res) {
      this.weather = res;
    },
    getTime(_timestamp) {
      let timezoneOffset = this.weather.timezone / 60;
      timezoneOffset = timezoneOffset / 60;
      console.log(timezoneOffset);
      let date = new Date(_timestamp * 1000);
      let utcDate = date.getUTCHours();
      let hours = 0;
      let minutes = 0;
      if (timezoneOffset % 1 === 0) {
        utcDate = utcDate === 0 ? 24 : utcDate;
        hours = utcDate + timezoneOffset;
        minutes = date.getUTCMinutes();
        console.log("~" + utcDate + " - " + timezoneOffset);
      } else {
        hours = utcDate + Math.trunc(timezoneOffset);
        minutes = date.getUTCMinutes() + 30;
        console.log("~~" + hours);
      }
      // let seconds = date.getSeconds();

      let ampm = hours >= 12 ? " PM" : " AM";
      hours = hours % 12;
      hours = hours ? hours : 12;
      minutes = minutes < 10 ? "0" + minutes : minutes;
      if (minutes >= 60) {
        minutes -= 60;
        hours += 1;
      }
      // seconds = seconds < 10 ? "0" + seconds : seconds;
      var strTime = hours + ":" + minutes + ampm;

      return strTime;
    }
  },
  created() {
    //do we support geolocation
    if (!("geolocation" in navigator)) {
      this.errorStr = "Geolocation is not available.";
      return;
    }

    this.gettingLocation = true;
    // get position
    navigator.geolocation.getCurrentPosition(
      pos => {
        this.gettingLocation = false;
        this.location = pos;
      },
      err => {
        this.gettingLocation = false;
        this.errorStr = err.message;
      }
    );
  },
  mounted() {
    // if (navigator.geolocation) {
    //   let loc = navigator.geolocation.getCurrentPosition();
    //   console.log(loc);
    // }
    this.query = "Rawalpindi";
    fetch(
      `${this.url_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}`
    )
      .then(res => {
        return res.json();
      })
      .then(this.setWeather);
  }
};
</script>

<style>
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}
body {
  font-family: "montserrat", sans-serif;
}
#app {
  background-image: url("https://source.unsplash.com/random/1600x900?nature,water,art,space,wild,happy,city,food,history,animal,car");
  background-size: cover;
  background-position: bottom;
  transition: 0.4s;
}
.welcome-heading {
  color: #fff;
  font-size: 32px;
  font-weight: 300;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
  margin-bottom: 15px;
}
main {
  min-height: 100vh;
  padding: 25px;
  background-image: linear-gradient(
    to bottom,
    rgba(0, 0, 0, 0.25),
    rgba(0, 0, 0, 0.75)
  );
}
.search-box {
  width: 100%;
  margin-bottom: 30px;
}
.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;

  color: #313131;
  font-size: 20px;
  appearance: none;
  border: none;
  outline: none;
  background: none;
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
}
.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}
.location-box .location {
  color: #fff;
  font-size: 32px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}
.location-box .date {
  color: #fff;
  font-size: 20px;
  font-weight: 300;
  font-style: italic;
  text-align: center;
}
.weather-box {
  text-align: center;
}
.weather-box .temp-box {
  display: inline-block;
  padding: 10px 25px;
  color: #fff;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.25);
  border-radius: 16px;
  margin: 30px 0px;
  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
.weather-box .temp-box .temp {
  font-size: 102px;
  font-weight: 900;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
.weather-box .temp-box .temp-desc {
  margin-top: 5px;
  font-size: 26px;
  font-weight: 500;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
.weather-box .weather {
  color: #fff;
  font-size: 48px;
  font-weight: 700;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
.weather-box .weather-desc {
  color: #fff;
  font-size: 26px;
  font-weight: 400;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
.change-background {
  position: fixed;
  bottom: 0;
  right: 0;
  color: gray;
  cursor: pointer;
}
</style>
