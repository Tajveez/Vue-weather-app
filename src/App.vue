<template>
  <div id="app">
    <main v-if="username && location">
      <div class="welcome-heading">Welcome, {{username}}</div>
      <div class="search-box">
        <input
          type="text"
          id="search-bar"
          class="search-bar"
          v-model="query"
          placeholder="What's on your Mind?"
          @keypress="search"
        />
      </div>
      <div class="weather-wrap" v-if="weather">
        <div class="location-box" v-if="weather.sys.country !== 'undefined'">
          <div class="location">
            <i class="fa fa-map-marker" aria-hidden="true"></i>
            {{ weather.name }}, {{ weather.sys.country }}
          </div>
          <div class="date">{{ getCurrentDate() }}</div>
          <!-- <div class="date">Monday, 20 July 2020</div> -->
        </div>
        <div class="weather-box">
          <div class="temp-box">
            <div class="temp">
              {{ weather.main.temp }}° C
              <hr />
            </div>
            <div class="temp-desc">
              <div style="font-size:12px">sunrise / sunset</div>
              <i class="fa fa-sun-o"></i>
              {{ getTime(weather.sys.sunrise) }} |
              <i class="fa fa-moon-o"></i>
              {{ getTime(weather.sys.sunset) }}
            </div>
          </div>
          <div class="weather">
            <!-- <img id="wicon" :src="weatherIcon" alt="Weather icon" /> -->
            {{ weather.weather[0].main }}
          </div>
          <div class="weather-desc">{{ weather.weather[0].description }}</div>
        </div>
      </div>
      <div class="settings">
        <a @click="setSetting">
          <i class="fa fa-cog"></i> Setting
        </a> |
        <a @click="fillCoffee">
          <i class="fa fa-coffee"></i> Support the developer
        </a> |
        <a @click="getContact">
          <i class="fa fa-github" aria-hidden="true"></i> contact
        </a>
      </div>
    </main>
    <main v-else>
      <div class="search-box">
        <div class="welcome-heading">
          Welcome
          <p style="font-size: 24px">Please, enter details to continue</p>
        </div>
        <input
          id="username"
          class="search-bar"
          type="text"
          name="username"
          :value="setting.username"
          placeholder="Enter your Name"
        />
        <br />
        <input
          id="location"
          class="search-bar"
          type="text"
          name="location"
          :value="setting.location"
          placeholder="Enter your Location (i.e. city name)"
        />
        <button @click="saveValues">Save</button>
      </div>
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
      username: "",
      location: "",
      weather: {},
      apiError: false,
      setting: {
        username: "",
        location: ""
      },
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
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday"
      ],

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
    setLocalStorage() {
      if (this.username !== "" && this.location !== "") {
        localStorage.setItem("username", this.username);
        localStorage.setItem("location", this.location);
      }
    },
    search(e) {
      if (e.key == "Enter" && this.query !== "") {
        window.open(
          "http://www.google.com/search?&q=" + escape(this.query),
          "_self"
        );
      }
    },
    fillCoffee() {
      window.open("https://www.patreon.com/tajveez");
    },
    getContact() {
      window.open("https://github.com/tajveez");
    },
    saveValues() {
      let username = document.querySelector("input[name=username]").value;
      let location = document.querySelector("input[name=location]").value;
      if (username === "" || location === "") {
        alert("Please, fill the values");
      } else {
        // console.log(username + " - " + location);
        this.getWeather(location).then(data => {
          if (data.cod != "404") {
            this.setWeather(data);
            this.username = username;
            this.location = location;
            this.setLocalStorage();
          } else {
            alert("Your entered location is invalid, please, try again.");
          }
        });
      }
    },
    setSetting() {
      let username = this.username;
      let location = this.location;

      this.username = "";
      this.location = "";
      this.setting.username = username;
      this.setting.location = location;

      // let url =
      //   "https://source.unsplash.com/random/1600x900?nature,water,art,space,wild,happy,city,food,history,animal,car";
      // let app = document.getElementById("app");
      // app.style.backgroundImage = `url('${url}')`;
    },
    getCurrentDate() {
      let today = new Date(this.weather.dt * 1000);
      let date =
        this.dayName[today.getDay()] +
        ", " +
        today.getDate() +
        " " +
        this.monthName[today.getMonth()] +
        " " +
        today.getFullYear();

      return date;
    },
    async getWeather(loc) {
      if (loc !== "") {
        let response = await fetch(
          `${this.url_base}weather?q=${loc}&units=metric&APPID=${this.api_key}`
        );
        let data = await response.json();
        return data;
      }
    },
    setWeather(res) {
      this.weather = res;
    },
    getTime(_timestamp) {
      let timezoneOffset = this.weather.timezone / 60;
      timezoneOffset = timezoneOffset / 60;

      let date = new Date(_timestamp * 1000);
      let utcDate = date.getUTCHours();
      let hours = 0;
      let minutes = 0;
      if (timezoneOffset % 1 === 0) {
        // console.log("~" + utcDate + " - " + timezoneOffset);
        utcDate = utcDate === 0 ? 24 : utcDate;

        if (timezoneOffset > 0) {
          hours = utcDate + timezoneOffset;
        } else {
          hours = utcDate + timezoneOffset;
          hours = hours < 0 ? 24 + hours : hours;
          // hours =
          //   utcDate < Math.abs(timezoneOffset)
          //     ? Math.abs(timezoneOffset)
          //     : utcDate + timezoneOffset;
        }

        minutes = date.getUTCMinutes();
      } else {
        hours = utcDate + Math.trunc(timezoneOffset);
        minutes = date.getUTCMinutes() + 30;
        // console.log("~~" + hours);
      }
      // let seconds = date.getSeconds();

      let ampm = hours >= 12 ? " PM" : " AM";
      hours = hours % 12;
      hours = hours ? hours : 12;
      if (minutes >= 60) {
        minutes -= 60;
        hours += 1;
      }
      minutes = minutes < 10 ? "0" + minutes : minutes;
      // console.log(hours + "~" + minutes);
      // seconds = seconds < 10 ? "0" + seconds : seconds;
      var strTime = hours + ":" + minutes + ampm;

      return strTime;
    }
  },
  created() {
    // if (navigator.geolocation) {
    //   let loc = navigator.geolocation.getCurrentPosition();
    //   console.log(loc);
    // }
    let username = localStorage.getItem("username");
    let location = localStorage.getItem("location");

    if (
      username !== "null" &&
      username !== "" &&
      location !== "null" &&
      location !== ""
    ) {
      this.username = username;
      this.location = location;
      // let location = this.location;
      this.getWeather(location).then(data => {
        if (data.cod != "404") {
          this.setWeather(data);
        }
      });
    } else {
      this.username = "";
      this.location = "";
    }
  },
  loadAfter() {
    document.getElementById("search-bar").focus();
  }
};
</script>

<style>
@import "./assets/font-awesome/css/font-awesome.min.css";
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}
body {
  font-family: "montserrat", sans-serif;
}
button {
  margin-top: 20px;
  font-size: 28px;
  font-weight: 700;
  color: rgba(0, 0, 0, 0.55);
  border-radius: 0px 15px;
  padding: 10px;
  border: none;
  background: rgba(255, 255, 255, 0.65);
  transition: 0.5s;
  float: right;
}
button:hover {
  border-radius: 15px 0px;
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
.settings {
  margin-right: 5px;
  position: fixed;
  bottom: 0;
  right: 0;
  color: gray;
  cursor: pointer;
}
</style>
