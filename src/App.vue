<template>
  <div id="app">
    <div id="start">
      <h1>Dart Score</h1>
      <form @submit.prevent="start">
        <p>
          <label for="addScore">Add score</label>
          <input type="checkbox" id="addScore" v-model="scoreCounter" />
        </p>
        <p v-for="player in players" :key="player.id">
          <input
            required
            type="text"
            :id="'player' + player.id"
            :placeholder="'Player ' + player.id"
            v-model="player.name"
          />
        </p>
        <p>
          <button
            id="add"
            @click.prevent="addPlayer"
            v-if="this.players.length < 5"
          >
            Add player
          </button>
          <button
            id="delete"
            @click.prevent="deletePlayer"
            v-if="this.players.length > 2"
          >
            Delete player
          </button>
        </p>
        <button type="submit">Start</button>
      </form>
    </div>

    <div id="game">
      <table
        border="1"
        cellpadding="0"
        cellspacing="0"
        :width="scoreCounter ? '75%' : '100%'"
      >
        <tr>
          <th @click.prevent="modalSettings" v-html="settingsIcon">
            {{ settingsIcon }}
          </th>
          <th
            v-for="player in players"
            :key="player.id"
            :class="{ focus: player.focus }"
            :data-id="player.id"
            @click.prevent="changeFocus"
          >
            {{ player.name }}
          </th>
        </tr>
        <tr v-for="(n, index) in 6" :key="index">
          <td>{{ n + 14 }}</td>
          <td
            v-for="player in players"
            :key="player.id"
            data-step="0"
            :data-dart="n + 14"
            :data-id="player.id"
            :class="{ focus: player.focus }"
            @click.prevent="addStep"
          >
            {{ step0 }}
          </td>
        </tr>
        <tr>
          <td v-html="bull">
            {{ bull }}
          </td>
          <td
            v-for="player in players"
            :key="player.id"
            data-step="0"
            data-dart="bull"
            :data-id="player.id"
            :class="{ focus: player.focus }"
            @click.prevent="addStep"
          >
            {{ step0 }}
          </td>
        </tr>
      </table>
      <div class="scores" v-if="scoreCounter">
        <div
          class="score"
          v-for="player in players"
          :key="player.id"
          :data-id="player.id"
          :class="{ focus: player.focus }"
          @click.prevent="changeFocus"
        >
          <p>{{ player.name }}</p>
          <p>Score: {{ player.score }}</p>
        </div>
      </div>
      <div class="modalBg" v-if="scoreCounterModal">
        <div class="modal">
          <p>Block this target to add score ?</p>
          <p>
            <span
              :data-dart="modalTargetDart"
              :data-player="modalTargetPlayer"
              @click.prevent="blocked"
              >Yes</span
            >
            <span
              :data-dart="modalTargetDart"
              :data-player="modalTargetPlayer"
              @click.prevent="canceled"
              >No</span
            >
          </p>
        </div>
      </div>
      <div class="modalBg" v-if="winner">
        <div class="modal">
          <p>Congratulation !<br />{{ winner }} won the game</p>
          <p>
            <span @click.prevent="reload">New game</span>
          </p>
        </div>
      </div>
      <div class="modalBg" v-if="settings">
        <div class="modal">
          <p>Restart the game ?</p>
          <p>
            <span @click.prevent="reload">Yes</span>
            <span @click.prevent="modalSettings">No</span>
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      scoreCounter: true,
      scoreCounterModal: false,
      modalTargetDart: 0,
      modalTargetplayer: 0,
      winner: "",
      settings: false,
      players: [
        {
          id: 1,
          name: "",
          score: 0,
          focus: false,
        },
        {
          id: 2,
          name: "",
          score: 0,
          focus: false,
        },
      ],
      settingsIcon: `
<svg aria-hidden="true" focusable="false" data-prefix="fas" data-icon="cog" class="svg-inline--fa fa-cog fa-w-16" role="img" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512"><path fill="currentColor" d="M487.4 315.7l-42.6-24.6c4.3-23.2 4.3-47 0-70.2l42.6-24.6c4.9-2.8 7.1-8.6 5.5-14-11.1-35.6-30-67.8-54.7-94.6-3.8-4.1-10-5.1-14.8-2.3L380.8 110c-17.9-15.4-38.5-27.3-60.8-35.1V25.8c0-5.6-3.9-10.5-9.4-11.7-36.7-8.2-74.3-7.8-109.2 0-5.5 1.2-9.4 6.1-9.4 11.7V75c-22.2 7.9-42.8 19.8-60.8 35.1L88.7 85.5c-4.9-2.8-11-1.9-14.8 2.3-24.7 26.7-43.6 58.9-54.7 94.6-1.7 5.4.6 11.2 5.5 14L67.3 221c-4.3 23.2-4.3 47 0 70.2l-42.6 24.6c-4.9 2.8-7.1 8.6-5.5 14 11.1 35.6 30 67.8 54.7 94.6 3.8 4.1 10 5.1 14.8 2.3l42.6-24.6c17.9 15.4 38.5 27.3 60.8 35.1v49.2c0 5.6 3.9 10.5 9.4 11.7 36.7 8.2 74.3 7.8 109.2 0 5.5-1.2 9.4-6.1 9.4-11.7v-49.2c22.2-7.9 42.8-19.8 60.8-35.1l42.6 24.6c4.9 2.8 11 1.9 14.8-2.3 24.7-26.7 43.6-58.9 54.7-94.6 1.5-5.5-.7-11.3-5.6-14.1zM256 336c-44.1 0-80-35.9-80-80s35.9-80 80-80 80 35.9 80 80-35.9 80-80 80z"></path></svg>`,
      bull: `<svg version="1.1" id="bull" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
	 viewBox="0 0 50 50" style="enable-background:new 0 0 50 50;" xml:space="preserve">
<style type="text/css">
	#bull .st0{fill:#FFFFFF;}
</style>
<g>
	<path class="st0" d="M30.61,41.26c-1.55-0.86-2.52-0.88-2.98-0.06c-0.35,0.63-0.05,1.82,0.62,2.08c1.34,0.51,1.51-0.69,1.95-1.47
		c0.68,0.85,0.3,2.25-0.6,3.68c-0.98,1.55-5.46,2.1-7.2,1.25c-1.97-0.97-3.13-3.25-2.6-5.77c0.27,0.6,0.37,1.03,0.63,1.32
		c0.34,0.38,0.91,0.99,1.19,0.9c0.51-0.16,1.09-0.69,1.26-1.2c0.22-0.67-0.26-1.22-1.01-1.37c-0.63-0.12-1.29-0.35-1.89-0.25
		c-0.96,0.16-1.29-0.33-1.72-1.01c-0.96-1.54-1.97-3.04-3.03-4.5c-1.36-1.87-1.61-5.58-0.29-7.31c2.02,2,4.38,3.3,7.38,3.05
		c-0.6-1.35-1.69-2.17-3.08-2.69c-1.06-0.39-2.11-0.8-3.13-1.28c-1.36-0.64-2.3-1.69-2.24-3.25c0.07-1.79,0.23-3.6,0.56-5.36
		c0.25-1.33,1.19-2.25,2.54-2.66c5.01-1.52,10.05-2.07,15.1-0.21c3.74,1.37,4.73,5.39,4.27,8.48c-0.28,1.91-1.83,2.53-3.35,3.17
		c-1.07,0.45-2.19,0.84-3.2,1.41c-0.92,0.52-1.63,1.3-1.68,2.55c2.67-0.41,4.99-1.37,6.88-3.34c1.15,0.54,1.61,1.61,1.43,2.67
		c-0.29,1.72-0.71,3.48-1.45,5.05c-0.69,1.47-1.86,2.72-2.83,4.06C31.65,39.89,31.12,40.57,30.61,41.26z"/>
	<path class="st0" d="M14.54,15.63c-0.98,2.45-2.39,4.62-2,7.35c-3.09-0.04-6.05-0.52-8.75-2c-3.58-1.96-4.64-5.34-3.11-9.66
		C1.64,8.59,3.4,6.53,5.96,5.23c1.74-0.88,3.57-1.58,5.36-2.36c0.05,0.08,0.11,0.16,0.16,0.23c-0.47,0.43-0.97,0.82-1.4,1.29
		C9.03,5.53,7.92,6.62,6.99,7.85c-1.43,1.9-1.23,3.89,0.42,5.62c1.45,1.52,3.34,1.99,5.33,2.15C13.3,15.67,13.85,15.63,14.54,15.63z
		"/>
	<path class="st0" d="M37.95,22.9c0.42-2.71-0.92-4.89-1.79-7.2c1.2-0.18,2.37-0.25,3.48-0.55c0.89-0.24,1.79-0.64,2.54-1.17
		c2.22-1.6,2.69-4.15,1.07-6.33c-1.28-1.72-2.8-3.25-4.06-4.69c6.42,1.82,10.07,4.94,10.75,11.08c0.3,2.67-0.64,4.83-2.86,6.35
		C44.34,22.27,41.19,22.71,37.95,22.9z"/>
</g>
</svg>`,
      step0: "",
      step1: `<svg version="1.1" id="step1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
	 viewBox="0 0 50 50" style="enable-background:new 0 0 50 50;" xml:space="preserve">
<style type="text/css">
	#step1 .st0{fill:none;stroke:#FFFFFF;stroke-width:3;stroke-miterlimit:10;}
</style>
<line class="st0" x1="2" y1="48" x2="48" y2="2"/>
</svg>`,
      step2: `<svg version="1.1" id="step2" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
	 viewBox="0 0 50 50" style="enable-background:new 0 0 50 50;" xml:space="preserve">
<style type="text/css">
	#step2 .st0{fill:none;stroke:#FFFFFF;stroke-width:3;stroke-miterlimit:10;}
</style>
<line class="st0" x1="2" y1="2" x2="48" y2="48"/>
<line class="st0" x1="2" y1="48" x2="48" y2="2"/>
</svg>`,
      step3: `<svg version="1.1" id="step3" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
	 viewBox="0 0 50 50" style="enable-background:new 0 0 50 50;" xml:space="preserve">
<style type="text/css">
	#step3 .st0{fill:none;stroke:#FFFFFF;stroke-width:3;stroke-miterlimit:10;}
</style>
<circle class="st0" cx="25" cy="25" r="22.08"/>
<line class="st0" x1="2" y1="2" x2="48" y2="48"/>
<line class="st0" x1="2" y1="48" x2="48" y2="2"/>
</svg>`,
    };
  },
  methods: {
    start() {
      document.querySelector("#start").remove();
      document.querySelector("#game").classList.add("show");
    },
    addPlayer() {
      this.players.push({
        id: this.players.length + 1,
        name: "",
        score: 0,
        focus: false,
      });
    },
    deletePlayer() {
      this.players.pop();
    },
    addStep(e) {
      let target = e.target;
      let index = target.getAttribute("data-id") - 1;

      if (this.players[index].focus) {
        if (target.getAttribute("data-step") === "0") {
          target.innerHTML = this.step1;
          target.setAttribute("data-step", "1");
        } else if (target.getAttribute("data-step") === "1") {
          target.innerHTML = this.step2;
          target.setAttribute("data-step", "2");
        } else if (target.getAttribute("data-step") === "2") {
          target.innerHTML = this.step3;
          target.setAttribute("data-step", "3");
        } else if (target.getAttribute("data-step") === "3") {
          if (target.getAttribute("data-blocked", true)) {
            this.players.filter((el) => {
              if (el.id !== Number(target.getAttribute("data-id"))) {
                if (
                  !document
                    .querySelector(
                      'td[data-id="' +
                        el.id +
                        '"][data-dart="' +
                        target.getAttribute("data-dart") +
                        '"]'
                    )
                    .hasAttribute("data-blocked")
                ) {
                  if (target.getAttribute("data-dart") === "bull") {
                    el.score += 25;
                  } else {
                    el.score += Number(target.getAttribute("data-dart"));
                  }
                }
              }
            });
          } else {
            if (this.scoreCounter) {
              this.scoreCounterModal = true;
              this.modalTargetDart = target.getAttribute("data-dart");
              this.modalTargetPlayer = target.getAttribute("data-id");
            } else {
              target.innerHTML = this.step0;
              target.setAttribute("data-step", "0");
            }
          }
        }

        this.checkWin();
      }
    },
    changeFocus(e) {
      let index;

      if (e.target.tagName === "TH" || e.target.tagName === "DIV") {
        index = e.target.getAttribute("data-id") - 1;
      } else if (e.target.tagName === "P") {
        index = e.target.parentNode.getAttribute("data-id") - 1;
      }

      if (this.players[index].focus) {
        this.players[index].focus = false;
      } else {
        this.players.filter((el) => {
          el.focus = false;
          this.players[index].focus = true;
        });
      }
    },
    blocked(e) {
      this.scoreCounterModal = false;
      let dataDart = e.target.getAttribute("data-dart");
      let dataPlayer = e.target.getAttribute("data-player");
      let el = document.querySelector(
        'td[data-dart="' + dataDart + '"][data-id="' + dataPlayer + '"]'
      );
      el.setAttribute("data-blocked", true);
      this.modalTargetDart = 0;
      this.modalTargetPlayer = 0;
      this.checkWin();
    },
    canceled(e) {
      this.scoreCounterModal = false;
      let dataDart = e.target.getAttribute("data-dart");
      let dataPlayer = e.target.getAttribute("data-player");
      let el = document.querySelector(
        'td[data-dart="' + dataDart + '"][data-id="' + dataPlayer + '"]'
      );
      el.innerHTML = this.step0;
      el.setAttribute("data-step", "0");
      this.modalTargetDart = 0;
      this.modalTargetPlayer = 0;
    },
    reload() {
      document.location.reload();
    },
    checkWin() {
      if (this.scoreCounter) {
        let result = this.players[0].score;
        let resultID;
        this.players.filter((el) => {
          if (el.score <= result) {
            result = el.score;
            resultID = el.id;
          }
        });
        if (
          result !== 0 &&
          document.querySelectorAll(
            'td[data-step="3"][data-blocked="true"][data-id="' + resultID + '"]'
          ).length === 7
        ) {
          this.winner = this.capitalizeFirstLetter(
            this.players[resultID - 1].name
          );
        } else {
          this.players.filter((el) => {
            if (
              el.score === result &&
              document.querySelectorAll(
                'td[data-step="3"][data-blocked="true"][data-id="' +
                  el.id +
                  '"]'
              ).length === 7
            ) {
              this.winner = this.capitalizeFirstLetter(el.name);
            }
          });
        }
      } else {
        this.players.filter((el) => {
          if (
            document.querySelectorAll(
              'td[data-step="3"][data-blocked="true"][data-id="' + el.id + '"]'
            ).length === 7
          ) {
            this.winner = this.capitalizeFirstLetter(el.name);
          }
        });
      }
    },
    capitalizeFirstLetter(string) {
      return string.charAt(0).toUpperCase() + string.slice(1);
    },
    modalSettings() {
      this.settings = !this.settings;
    },
  },
};
</script>

<style lang="scss">
body {
  background-color: black;
  color: white;
  padding: 0.5rem;
  font-family: sans-serif;
  #app {
    #start {
      border: 1px solid white;
      width: 75%;
      margin: 0 auto;
      padding: 1rem;
      text-align: center;
      font-size: 1rem;
      h1 {
        margin: 0;
        text-transform: uppercase;
        font-size: 2rem;
      }
      form {
        p:first-of-type {
          display: flex;
          justify-content: center;
          align-items: center;
          input {
            margin: 0 0 0 0.5rem;
          }
        }
        input {
          background: transparent;
          border: 1px solid white;
          outline: none;
          box-shadow: none;
          color: white;
          padding: 0.5rem;
          text-transform: capitalize;
        }
        button {
          margin: 0 0.5rem;
          padding: 0.5rem;
          border: 1px solid white;
          color: white;
          cursor: pointer;
          &#add {
            border: 1px solid green;
            background: green;
          }
          &#delete {
            border: 1px solid red;
            background: red;
          }
          &[type="submit"] {
            font-size: 1.2rem;
            text-transform: uppercase;
            margin-top: 0.5rem;
          }
        }
      }
    }
    #game {
      display: none;
      justify-content: space-between;
      table {
        table-layout: fixed;
        th {
          padding: 0.5rem 0.25rem;
          text-transform: capitalize;
          &.focus {
            color: orange;
            border-color: orange;
          }
          &:first-of-type {
            width: 2rem;
          }
          svg {
            width: 1rem;
            vertical-align: middle;
          }
        }
        td {
          text-align: center;
          padding: 0.5rem 0.25rem;
          position: relative;
          &.focus {
            border-color: orange;
            svg * {
              stroke: orange;
            }
          }
          &::after {
            content: "";
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: transparent;
          }
          &[data-blocked] {
            svg * {
              stroke: red;
            }
          }
          svg {
            width: 1rem;
            vertical-align: middle;
            circle {
              fill: none;
            }
          }
        }
      }
      .scores {
        width: 25%;
        margin-left: 1rem;
        .score {
          border: 1px solid white;
          padding: 0.5rem;
          &.focus {
            color: orange;
            border-color: orange;
          }
          &:not(:last-of-type) {
            margin-bottom: 0.5rem;
          }
          p {
            margin: 0;
            &:first-of-type {
              margin-bottom: 0.25rem;
              text-transform: capitalize;
            }
          }
        }
      }
      .modalBg {
        position: fixed;
        top: 0;
        left: 0;
        background-color: rgba(0, 0, 0, 0.5);
        width: 100%;
        height: 100%;
        text-align: center;
        .modal {
          display: inline-flex;
          flex-direction: column;
          background-color: black;
          border: 1px solid white;
          padding: 1rem;
          margin-top: 3rem;
          width: 50%;
          line-height: 1.5;
          span {
            border: 1px solid white;
            padding: 0.5rem;
            margin: 0 0.5rem;
            min-width: 2rem;
            display: inline-flex;
            justify-content: center;
            cursor: pointer;
          }
        }
      }
      &.show {
        display: flex;
      }
    }
  }
}
</style>
