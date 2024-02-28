<template>
  <div id="body" class="dashboard">
    <h1 class="text" v-if="showProcess3" id="mainTitle">{{ msg }}</h1>
   <p class= "maindiv" v-if="showProcess2" id="messageTwo">
     <section  class="text"> {{ msg2 }}</section>
      <span class="text" v-if="showEyes"
        ><input
          id="variableOneInput"
          type="input"
          v-model="variableOne"
          placeholder="liberal?"
        /><br>
        eyes</span
      ><br />
    </p>
    <p id="terminal"></p>
    <section class="images" id="image1"></section>
    <section class="images" id="image2"></section>
    <p v-if="showProcess" id="messageThree">
      {{ msg3 }}
    </p>
    <p id="results">
      <span id="overallMoralFoundations"></span
      ><span id="overallEmotions"></span>
    </p>
    <section v-if="!showProcess2" id="overalExplanation">
      {{ overallOutputExplanation }} {{ overallSummaryOutput }}
    </section>
    <!-- <input
      id="APIinput"
      type="text"
      v-model="apiKEY"
      v-if="showAPI"
      placeholder="Enter OpenAI API Key"
    /><br /><span v-if="showAPI"
      >Set up an OpenAI account and get an API key
      <a href="https://openai.com/product">here</a>.</span
    >
    <br /><button v-if="showAPI" id="apiButton" @click="registerAPI">
      Set API Key
    </button> -->
    
    <br /><br />
    
    <input
      v-if="showProcess"
      id="URLInput"
      type="input"
      v-model="urlToScrape"
      placeholder="Enter URL to Imagine"
    />
    <p id="status">
      <span v-if="!reveal" class="titleIsh">Image Prompt: </span>{{ status }}
    </p>
    <button v-if="showProcess" id="startButton" @click="checkForGoodToGo">
      Visualize
    </button>
    <p id="pageText">
      <span v-if="!reveal2" class="titleIsh">Original text: </span
      >{{ pageText }}
    </p>
    <br /><button v-if="!showPrint" id="apiButton" @click="pdfResults">
      Save Results as PDF
    </button>
    <p v-if="!showProcess"> <a href="https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExZWM2dHh5aHpkejgzZjFsd2NldjA5MndlbjR3MW0wdXB1ZG1reDBoZCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/i2tUkY5YrfN85y1W0P/giphy.gif">Loading Image Credit</a></p>

    <!-- <button @click="renderVisuals">Overall</button> 

   
    <button @click="getEmotionStats">Analyze Emotion</button>
    
    <button @click="returnJSON">Return Usable JSON</button>
    <button @click="renderVisuals">Visualize</button>
 -->

    <section id="visuals" class="visuals"></section>
    <section id="specificAnalysis"></section>
    <section id="specificAnalysis2"></section>
    <section id="specificAnalysis3"></section>
    <section id="specificAnalysis4"></section>
    <section id="specificAnalysis5"></section>
  </div>
</template>

<script>
//import paralleldots from "paralleldots";
import * as rs from "text-readability";
//import * as cheerio from 'cheerio';
import axios from "axios";
import Plotly from "plotly.js-dist";
import dotenv from "dotenv";
dotenv.config();
//import OpenAI from "openai";
export default {
  name: "imagination_window",
  props: {},
  data() {
    return {
      msg: "Symbolic Synthesizer",
      msg2: "See the world through ",
      msg3: "",
      variableOne: "",
      showEyes: true,
      urlToScrape: "https://www.",
      pageText: "",
      anger: 0,
      fear: 0,
      surprise: 0,
      disgust: 0,
      sadness: 0,
      happiness: 0,
      readability: 0,
      loading: true,
      showProcess: true,
      showProcess2: true,
      showProcess3: true,
      showAPI: true,
      anchorsForCrawl: [],
      secondIteration: false,
      JSON1: null,
      JSON2: null,
      JSON3: null,
      JSON4: null,
      moralFoundationAnalysis: "",
      unique: [],
      overallSummaryOutput: "",
      overallOutputExplanation: "",
      showPrint: true,
      reveal: true,
      reveal2: true,
      apiKEY: process.env.VUE_APP_ROOT_API_KEY,
      goodToGo: false, 
      status: "Enter a worldview and url above. Mission statements and about pages work best."
    };
  },

  created: function () {},

  methods: {
    registerAPI: function () {
      this.showAPI = false;
    },

    checkForGoodToGo: function () {
     if (this.urlToScrape.endsWith("$")) {
      this.goodToGo = true; 
      this.urlToScrape = this.urlToScrape.slice(0, -1);
      this.grabPage();
     }
     else {
      this.goodToGo = false;
      this.msg = "Please enter a valid url"
     }
    }, 

    grabPage: function () {
        if (this.goodToGo == false) {
          this.msg = "Please enter a valid url"
        }
      if (this.goodToGo == true) {
      this.showProcess = false;
      this.showEyes = false;
      let img = document.createElement("img");
      img.src =
        "https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExZWM2dHh5aHpkejgzZjFsd2NldjA5MndlbjR3MW0wdXB1ZG1reDBoZCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/i2tUkY5YrfN85y1W0P/giphy.gif";
      img.setAttribute("id", "thinkingIMG");
      img.style.width = "30%";
      document.getElementById("image2").appendChild(img);
      this.msg2 = "Extracting text...";
      this.status = "";
      if (!this.urlToScrape.endsWith("/")) {
        this.urlToScrape =
          "https://api.allorigins.win/raw?url=" +
          encodeURIComponent(this.urlToScrape + "/");
        console.log("slash added");
      }
      if (this.urlToScrape.endsWith("/")) {
        this.urlToScrape =
          "https://api.allorigins.win/raw?url=" +
          encodeURIComponent(this.urlToScrape);
        console.log("slash not added");
      }

      const url = this.urlToScrape;
      axios
        .get(url)
        .then((response) => {
          const data = response.data;
          const data2 = data.replace(/\s+/g, " ").trim();
          const parser = new DOMParser();
          const html = parser.parseFromString(data2, "text/html");
          const workingHTML = html;
          workingHTML
            .querySelectorAll("script, style")
            .forEach((s) => s.remove());
          let htmlWithoutScripts = workingHTML
            .querySelector("body")
            .innerText.trim();

          if (this.secondIteration == false) {
            var anchors = [],
              l = html.links;
            const tickerA = html.links.length;
            for (var i = 0; i < l.length; i++) {
              const counterTickerA = i;
              if (html.links[i].href.includes(this.urlToScrape)) {
                anchors.push(l[i].href);
              }
              if (counterTickerA === tickerA - 1) {
                this.reveal2 = false;
                this.pageText = htmlWithoutScripts;

                if (!this.pageText.endsWith(".")) {
                  this.pageText = this.pageText + ".";
                  console.log("ngtg");
                }

                if (this.pageText.endsWith(".")) {
                  console.log("gtg");
                }
                this.summarizeText();
              }
            }
          }
        })
        .catch((errors) => {
          console.log(errors);
          this.msg = errors; // Errors and stuff
        });

        } 
    },

    summarizeText: function () {
      this.msg2 = "Generating image prompt from text...";
      const client = axios.create({
        headers: {
          Authorization: "Bearer " + this.apiKEY,
        },
      });

      const params = {
        model: "gpt-3.5-turbo-16k",
        messages: [
          {
            role: "user",
            content:
              "Write a prompt for dall-e-3 based on the themes and content of this text. Text: " +
              this.pageText,
          },
        ],
        temperature: 0,
        max_tokens: 256,
        top_p: 1,
        frequency_penalty: 0,
        presence_penalty: 0,
      };
      //test
      client
        .post("https://api.openai.com/v1/chat/completions", params)
        .then((result) => {
          const rawResult = result.data.choices[0].message.content;
          const rawResult2 = rawResult.replaceAll(
            "Create a prompt for dall-e-3: ",
            ""
          );
          this.reveal = false;
          this.status = rawResult2.replaceAll('"', "");
          this.status = this.status +
          " Render the image as if through " +
          this.variableOne +
          " eyes. Do not attempt to include text."
          this.getImageBasedOnText();
        })
        .catch((error) => {
          console.log(error);
          this.msg = "first call " + error;
        });
    },

    getImageBasedOnText: function () {
      this.msg2 = "Generating image from prompt....";
      const client = axios.create({
        headers: {
          Authorization: "Bearer " + this.apiKEY,
        },
      });
      //adds playmeow
      //playMeowSound: function ()
      //{
      //playMeowSound: function () {
      //const meowSound = new Audio('\project\imaginationwindow\public\cat-meow-6226.mp3');
      //meowSound.play().catch(error => console.error('Error playing sound:', error));
      //},
      const params = {
        model: "dall-e-3",
        prompt:
          this.status,
        n: 1,
        size: "1024x1024",
      };

      client
        .post("https://api.openai.com/v1/images/generations", params)
        .then((result) => {
          document.getElementById("thinkingIMG").remove();
          this.imageURL1 = result.data.data[0].url;
          
          var div = document.getElementById("image1");
          var p = document.createElement("img");
          p.style.width = "75%";
          p.src = this.imageURL1;
          div.append(p);
          //this.playMeowSound();
          this.msg2 =
            "Process complete. Image, prompt, and starting text below.";
        })
        .catch((error) => {
          console.log("image 1" + error);
          this.msg = "image 2" + error;
        });
    },

    getMoralFoundations: function () {
      var workingJSON = document.getElementById("specificAnalysis2").innerText;
      const middleJSON2 = "[" + workingJSON.slice(0, -1) + "]";
      const workingJSON2 = JSON.parse(middleJSON2);
      let i3,
        len3 = workingJSON2.length;
      const ticker3 = workingJSON2.length;
      const instance = this;

      for (i3 = 0; i3 < len3; i3++) {
        fire(i3);
      }

      function fire(i3) {
        setTimeout(function () {
          console.log("Analyzing moral foundations");

          const usableURL2 = workingJSON2[i3].url;
          const counterTicker3 = i3;
          const angry = workingJSON2[i3].anger;
          const happy = workingJSON2[i3].happiness;
          const disgusted = workingJSON2[i3].disgust;
          const fearful = workingJSON2[i3].fear;
          const surprised = workingJSON2[i3].surprise;
          const sad = workingJSON2[i3].sadness;
          const usableText2 = workingJSON2[i3].text;
          const pageType2 = "subPage";
          const client = axios.create({
            headers: {
              Authorization: "Bearer " + instance.apiKEY,
            },
          });

          const params = {
            model: "gpt-3.5-turbo-instruct",
            prompt:
              "Analyze this text to identify which of these five moral foundations that it best represents: care, fairness, loyalty, authority, and purity. Include an explanation. Text:" +
              usableText2 +
              ".",
            temperature: 0,
            max_tokens: 275,
            top_p: 1,
            frequency_penalty: 0,
            presence_penalty: 0,
          };

          client
            .post("https://api.openai.com/v1/completions", params)
            .then((result) => {
              instance.msg = "Analyzing moral foundations of";
              instance.msg2 = usableURL2;
              console.log(result.data.choices[0].text);
              const moralFoundationResults =
                result.data.choices[0].text.replaceAll('"', "");
              instance.moralFoundationAnalysis =
                moralFoundationResults.replaceAll("'", "");

              var div = document.getElementById("specificAnalysis3");
              var p = document.createElement("div");
              p.innerHTML =
                '{"pageType":' +
                '"' +
                pageType2 +
                '"' +
                "," +
                '"url":' +
                '"' +
                usableURL2 +
                '"' +
                "," +
                '"text":' +
                '"' +
                usableText2 +
                '"' +
                "," +
                '"moralFoundation":' +
                '"' +
                instance.moralFoundationAnalysis +
                '"' +
                "," +
                '"anger":' +
                angry +
                "," +
                '"fear":' +
                fearful +
                "," +
                '"surprise":' +
                surprised +
                "," +
                '"disgust":' +
                disgusted +
                "," +
                '"sadness":' +
                sad +
                "," +
                '"happiness":' +
                happy +
                "},";
              div.appendChild(p);
            })
            .catch((error) => {
              console.log(error);
              this.msg = error;
            });
          if (counterTicker3 === ticker3 - 1) {
            setTimeout(() => {
              console.log("Delayed for 5 seconds.");
              instance.returnJSON();
            }, 2000);
          }
        }, 500 * i3); //timeout
      } //fire
    },

    returnJSON: function () {
      var workingJSON = document.getElementById("specificAnalysis3").innerText;
      var middleJSON = "[" + workingJSON.slice(0, -1) + "]";
      this.JSON3 = middleJSON;
      console.log("JSON2: " + this.JSON3);
      var div = document.getElementById("specificAnalysis4");
      var p = document.createElement("div");
      p.innerHTML = this.JSON3;
      div.appendChild(p);

      setTimeout(() => {
        console.log("Delayed for 1 second.");
        this.renderVisuals();
      }, 2000);
    },

    renderVisuals: function () {
      document.getElementById("visuals").style.display = "block";
      document.getElementById("thinkingIMG").remove();

      let img = document.createElement("img");
      img.src = "https://media.giphy.com/media/QIRDfKwRFXz6nBCQkF/giphy.gif";
      img.setAttribute("id", "thinkingIMG2");
      document.getElementById("terminal").appendChild(img);
      var workingJSON = document.getElementById("specificAnalysis4").innerText;
      this.JSON4 = JSON.parse(workingJSON);

      //const usableText = JSON.stringify(this.JSON1[0].text);

      if (this.JSON4 != null) {
        console.log("test");
        var i,
          len = this.JSON4.length;
        for (i = 0; i < len; i++) {
          const usableURL = this.JSON4[i].url;
          const angry = this.JSON4[i].anger;
          const happy = this.JSON4[i].happiness;
          const disgusted = this.JSON4[i].disgust;
          const fearful = this.JSON4[i].fear;
          const surprised = this.JSON4[i].surprise;
          const sad = this.JSON4[i].sadness;
          const moralAnalysis = this.JSON4[i].moralFoundation;

          var div = document.getElementById("visuals");
          var p = document.createElement("div");
          p.innerHTML =
            "<h2>" +
            usableURL +
            "</h2>" +
            "<h3>Moral Foundations Analysis: </h3>" +
            moralAnalysis +
            "<h3>Emotional Analysis: </h3><ul>" +
            "<li> Anger: " +
            angry +
            "</li>" +
            "<li> Fear: " +
            fearful +
            "</li>" +
            "<li> Surprise: " +
            surprised +
            "</li>" +
            "<li> Disgust: " +
            disgusted +
            "</li>" +
            "<li> Sadness: " +
            sad +
            "</li>" +
            "<li> Happiness: " +
            happy +
            "</li>" +
            "</ul>";
          div.appendChild(p);

          if (i === len - 1) {
            setTimeout(() => {
              console.log("Delayed for 4 seconds.");
              document.getElementById("thinkingIMG2").remove();
              this.msg = "Analysis Complete";
              this.msg2 = "";
              this.getOverallMoralFoundationScores();
            }, 4000);
          }
        }
      }
    },

    getOverallMoralFoundationScores: function () {
      this.showProcess2 = false;
      this.msg = this.urlToScrape;
      var workingJSON = document.getElementById("specificAnalysis4").innerText;
      const test = JSON.parse(workingJSON);
      const instance = this;
      const pageType3 = "overallMoral";
      let overallMoralAnalysis = "";

      const e = test.length;
      for (var i = 0; i < e; i++) {
        overallMoralAnalysis = overallMoralAnalysis +=
          test[i].moralFoundation.substring(0, 70) + ".";
        if (i === e - 1) {
          instance.overallSummaryOutput = overallMoralAnalysis;

          const client = axios.create({
            headers: {
              Authorization: "Bearer " + instance.apiKEY,
            },
          });

          const params = {
            model: "gpt-3.5-turbo-instruct",
            prompt:
              'Give me scores between 1 and 100 that indicate the mentions of care, fairness, loyalty, authority, and purity in the following text formatted in JSON as {"care": number score,"fairness": number score,"loyalty": number score,"authority": number score,"purity": number score}. Explain those scores. Text:' +
              instance.overallSummaryOutput +
              ".",
            temperature: 0,
            max_tokens: 850,
            top_p: 1,
            frequency_penalty: 0,
            presence_penalty: 0,
          };

          client
            .post("https://api.openai.com/v1/completions", params)
            .then((result) => {
              instance.msg = instance.urlToScrape;
              const rawResultA = result.data.choices[0].text;
              console.log(rawResultA);
              const justTheTextA = rawResultA.substring(
                rawResultA.indexOf("}") + 1
              );
              const justTheJSONA = rawResultA.substring(
                0,
                rawResultA.indexOf("}") + 1
              );
              console.log(justTheTextA + " : " + justTheJSONA);
              const moralResultsA = JSON.parse(justTheJSONA);
              const overallCare = moralResultsA.care;
              const overallFairness = moralResultsA.fairness;
              const overallLoyalty = moralResultsA.loyalty;
              const overallAuthority = moralResultsA.authority;
              const overallPurity = moralResultsA.purity;
              instance.overallOutputExplanation = justTheTextA;
              //const moralFoundationResults3 = justTheTextA;
              // justTheTextA.text.replaceAll('"', "");
              // 					instance.moralFoundationAnalysis = moralFoundationResults3.replaceAll("'", "");
              // 					instance.overallSummaryOutput = instance.moralFoundationAnalysis

              var div = document.getElementById("specificAnalysis3");
              var p = document.createElement("div");
              p.innerHTML =
                '{"pageType":' +
                '"' +
                pageType3 +
                '"' +
                "," +
                '"url":' +
                '"' +
                instance.urlToScrape +
                '"' +
                "," +
                '"overallCare":' +
                overallCare +
                "," +
                '"overallFairness":' +
                overallFairness +
                "," +
                '"overallLoyalty":' +
                overallLoyalty +
                "," +
                '"overallAuthority":' +
                overallAuthority +
                "," +
                '"overallPurity":' +
                overallPurity +
                "," +
                '"moralFoundationOverall":' +
                '"' +
                justTheTextA +
                '"' +
                "},";
              div.appendChild(p);

              var data = [
                {
                  x: ["Care", "Fairness", "Loyalty", "Authority", "Purity"],
                  y: [
                    overallCare,
                    overallFairness,
                    overallLoyalty,
                    overallAuthority,
                    overallPurity,
                  ],
                  type: "bar",
                  marker: {
                    color: [
                      "#bf4e30",
                      "#c6ccb2",
                      "#93ff96",
                      "#e5eafa",
                      "#78fecf",
                    ],
                  },
                },
              ];

              var layout = {
                height: 450,
                width: 450,
                showlegend: false,
                paper_bgcolor: "#2b2d42",
                plot_bgcolor: "#2b2d42",
                title: "Moral Foundations",
                font: {
                  family: "Product Sans, monospace",
                  size: 25,
                  color: "white",
                },
                margin: {
                  l: 50,
                  r: 50,
                  b: 100,
                  t: 100,
                  pad: 4,
                },
              };
              var config = { responsive: true };
              Plotly.newPlot("overallMoralFoundations", data, layout, config);
              setTimeout(() => {
                console.log("Delayed for 1 second.");
                this.renderOverallEmotion();
              }, 1000);
            })
            .catch((error) => {
              console.log(error);
              this.msg = error;
            });
        }
      }
    },

    renderOverallEmotion: function () {
      //var workingJSON =
      //this.JSON4 = JSON.parse(workingJSON);
      var workingJSONB = document.getElementById("specificAnalysis4").innerText;
      const test = JSON.parse(workingJSONB);
      let overallAnger = 0;
      let overallFear = 0;
      let overallHappiness = 0;
      let overallSurprise = 0;
      let overallSadness = 0;
      let overallDisgust = 0;
      console.log(test.length);
      const instance = this;
      const e = test.length;
      for (var i = 0; i < e; i++) {
        overallAnger = overallAnger + test[i].anger;
        overallFear = overallFear + test[i].fear;
        overallHappiness = overallHappiness + test[i].happiness;
        overallSurprise = overallSurprise + test[i].surprise;
        overallSadness = overallSadness + test[i].sadness;
        overallDisgust = overallDisgust + test[i].disgust;
        if (i === e - 1) {
          overallAnger = Math.round(overallAnger / e) * 10;
          overallFear = Math.round(overallFear / e) * 10;
          overallHappiness = Math.round(overallHappiness / e) * 10;
          overallSurprise = Math.round(overallSurprise / e) * 10;
          overallSadness = Math.round(overallSadness / e) * 10;
          overallDisgust = Math.round(overallDisgust / e) * 10;

          var data = [
            {
              type: "pie",
              values: [
                overallAnger,
                overallFear,
                overallHappiness,
                overallSurprise,
                overallSadness,
                overallDisgust,
              ],
              labels: [
                "Anger",
                "Fear",
                "Happiness",
                "Surprise",
                "Sadness",
                "Disgust",
              ],
              textinfo: "label+percent",
              textposition: "outside",
              marker: {
                colors: [
                  "#ff0022",
                  "#ffbc42",
                  "#0496ff",
                  "#694d75",
                  "#1b5299",
                  "#40434e",
                ],
              },
            },
          ];

          var layout = {
            height: 450,
            width: 450,
            showlegend: false,
            paper_bgcolor: "#2b2d42",
            title: "Emotions",
            font: {
              family: "Arial, monospace",
              size: 25,
              color: "white",
            },
            margin: {
              l: 50,
              r: 50,
              b: 100,
              t: 100,
              pad: 4,
            },
          };
          var config = { responsive: true };
          Plotly.newPlot("overallEmotions", data, layout, config);

          setTimeout(() => {
            console.log("Delayed for 1 second.");
            instance.returnJSONAgain();
          }, 1000);
        }
      }
    },

    returnJSONAgain: function () {
      var workingJSON = document.getElementById("specificAnalysis3").innerText;
      var middleJSON = "[" + workingJSON.slice(0, -1) + "]";
      const middleWorkingJSON = middleJSON;
      console.log("JSON2: " + middleWorkingJSON);
      var div = document.getElementById("specificAnalysis5");
      var p = document.createElement("div");
      p.innerHTML = middleWorkingJSON;
      div.appendChild(p);
      this.showPrint = false;
    },

    getReadabilityStats: function () {
      this.readability = rs.gunningFog(this.workingOutput);
      console.log(this.readability + " " + this.workingOutput);
    },

    reset: function () {
      location.reload();
    },

    unhideData: function () {
      document.getElementById("rawData").style.display = "inline-block";
    },

    hideData: function () {
      document.getElementById("rawData").style.display = "none";
    },

    renderData: function () {
      const promise1 = new Promise((resolve, reject) => {
        this.getEmotionStats();
        this.getReadabilityStats();
        resolve("Data rendered!");
        reject("Data render failed");
      });

      promise1.then(() => {
        this.constructJSON();
        this.visualizeData();
        this.resetWorkingOutput();
        console.log("JSON and charts constructed");
      });
    },

    constructJSON: function () {
      this.currentDataObject =
        '{"time":' +
        '"' +
        this.workingTime +
        '"' +
        "," +
        '"wpm":' +
        '"' +
        this.wpm +
        '"' +
        "," +
        '"content":' +
        '"' +
        this.workingOutput +
        '"' +
        "," +
        '"Angry":' +
        this.anger +
        "," +
        '"Fear":' +
        this.fear +
        "," +
        '"Excited":' +
        this.excitement +
        "," +
        '"Bored":' +
        this.boredom +
        "," +
        '"Sad":' +
        this.sadness +
        "," +
        '"Happy":' +
        this.happiness +
        "," +
        '"volume":' +
        this.volumeValue +
        "," +
        '"readability":' +
        this.readability +
        "," +
        '"faceAnger":' +
        this.faceAngry +
        "," +
        '"faceDisgust":' +
        this.faceDisgusted +
        "," +
        '"faceFear":' +
        this.faceFearful +
        "," +
        '"faceHappiness":' +
        this.faceHappy +
        "," +
        '"faceNeutral":' +
        this.faceNeutral +
        "," +
        '"faceSadness":' +
        this.faceSad +
        "," +
        '"faceSurprise":' +
        this.faceSurprised +
        "},";
      var div = document.getElementById("rawData");
      div.innerHTML += this.currentDataObject;
      this.overallDataObject = document.getElementById("rawData").innerHTML;
    },

    pdfResults: function () {
      window.print();
    },
  }, //
}; //
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import url("https://fonts.cdnfonts.com/css/lcd");
#textEmotion,
#faceEmotion,
#voiceEmotion,
#wpm {
  display: inline-block;
}

#overallEmotions {
  display: inline-block;
}

#overalExplanation {
  color: orange;
  font-size: 25px;
  margin-top: 25px;
  width: 65%;
  display: inline-block;
}

#terminal {

}

#thinkingIMG {


}

#results {
  display: flex;
  margin: auto;
  width: 65%;
}

#variableOneInput {
  width: 200px;
  text-align: center;
   background: url("https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExa2R6ODR2Z3BvdjVqNjNtODY5Z2gycDd2ZjZmZzMydWN5ampqMWx6dSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/esMcTxGMz3Hhu/giphy.gif");
  background-size: cover;
  -webkit-background-clip: text;
  font-size: 35px;
  font-weight: bold;
  color: white; 
  border-radius: 10px; 
}
#overallMoralFoundatations {
  display: inline-block;
  margin: auto;
  display: none;
}

#URLInput {
  width: 40%;
  font-size: 25px;
  text-align: left;
  background-color: #CF9FFF;
  color: #252627;
  border: none;
  padding-left: 5%; 
}

#APIinput {
  width: 50%;
  font-size: 20px;
  text-align: center;
  background-color: hotpink;
  color: #252627;
  border: none;
}
#startButton {
  background: orange;
  font-size: 30px;
  color: black;
    border: none;

}

#startButton:hover {
  background: #007ba7;
}

#apiButton {
  background: #2f4858;
  font-size: 20px;
  color: white;
  border: none;
}

#apiButton:hover {
  background: purple;
}
.visuals {
  color: #ff66d8;
  border: solid;
  font-size: 20px;
  width: 100%;
  padding: 10px;
  display: none;
}

#specificAnalysis {
}
#specificAnalysis2 {
  color: pink;
}
#specificAnalysis3 {
  color: blue;
}
#specificAnalysis5 {
  color: orange;
  font-size: 20px;
}

#specificAnalysis4 {
  color: limegreen;
}
div {
  background-color: none;
  color: #71c68b;
}
.chartWindow {
  position: relative;
  display: inline-block;
  width: 80%;
}
.optionsButton {
  height: 50px;
  width: 75px;
  padding: 12px;
  font-size: 10px;
  margin: 5px;
  border: none;
  font-weight: bold;
  color: black;
  font-family: Arial, sans-serif;
}

.title {
  color: white;
}

#messageTwo {
  color: #92dce5;
  font-size: 25px;
}

#messageThree {
  color: white;
  font-size: 25px;
  margin-left: 10%;
}

#begin {
  background-color: #c300ff;
  border: none;
  height: 50px;
  width: 100px;
  font-weight: bold;
  color: black;
  font-family: Arial, sans-serif;
  font-size: 20px;
  margin-top: 40px;
  margin-bottom: -20px;
}

#begin:hover {
  background-color: #fdfd96;
}

#start {
  background-color: #cbc3e3;
  border: none;
  height: 50px;
  width: 100px;
  font-weight: bold;
  color: black;
  font-family: Arial, sans-serif;
  font-size: 20px;
  margin: 10px;
}

#start:hover {
  background-color: lightgreen;
}

#stop {
  background-color: #cbc3e3;
  border: none;
  height: 50px;
  width: 100px;
  font-weight: bold;
  color: black;
  font-family: Arial, sans-serif;
  font-size: 20px;
  margin: 10px;
}

#stop:hover {
  background-color: #ff726f;
}

#reset {
  background-color: #cbc3e3;
  border: none;
  height: 50px;
  width: 100px;
  font-weight: bold;
  color: black;
  font-family: Arial, sans-serif;
  font-size: 20px;
  margin: 10px;
}

#reset:hover {
  background-color: lightyellow;
}

#pdf {
  background-color: #c300ff;
  border: none;
  height: 50px;
  width: 100px;
  font-weight: bold;
  color: black;
  font-family: Arial, sans-serif;
  font-size: 20px;
  margin: 10px;
}

#pdf:hover {
  background-color: #00ffc3;
}

#next {
  background-color: #7766c6;
  border: none;
  height: 50px;
  width: 100px;
  font-weight: bold;
  color: black;
  font-family: Arial, sans-serif;
  font-size: 20px;
  margin-top: 40px;
  margin-bottom: -20px;
}

#next:hover {
  background-color: #ffc300;
}

#output {
  margin: auto;
  color: #f48d79;
  background-color: #222831;
  width: 80%;
  text-align: left;
  overflow: auto;
  height: 170px;
  font-size: 25px;
  margin-top: 0px;
  margin-bottom: 0px;
}

#wpmChart {
  overflow: auto;
  width: 80%;
  display: inline-block;
  margin-top: 3px;
  margin-bottom: 0px;
}

#readabilityChart {
  overflow: auto;
  width: 80%;
  display: inline-block;
  margin-top: 3px;
  margin-bottom: 0px;
}

#volumeChart {
  overflow: auto;
  width: 80%;
  display: inline-block;
  margin-top: -3px;
}

#textEmotionChart {
  overflow: auto;
  width: 80%;
  display: inline-block;
  margin-top: -3px;
}

#faceEmotionChart {
  overflow: auto;
  width: 80%;
  display: inline-block;
  margin-top: -3px;
}

#rawData {
  margin: auto;
  color: lawngreen;
  background-color: #222831;
  width: 80%;
  text-align: left;
  overflow: scroll;
  height: 100px;
  font-size: 25px;
  margin: 0px;
}

#dataHideButton {
  margin: auto;
  color: #222831;
  background-color: #222831;
  width: 40%;
  text-align: center;
  height: 30px;
  font-size: 10px;
  margin: 0px;
  border: none;
}

#dataShowButton {
  margin: auto;
  color: #222831;
  background-color: #222831;
  width: 40%;
  text-align: center;
  height: 30px;
  font-size: 10px;
  margin: 0px;
  border: none;
}

h1 {
  font-size: 50px;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0px;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

#talking {
  height: 100px;
  margin-bottom: -20px;
  -webkit-filter: invert(1);
  filter: invert(1);
}

#timer {
  background: #222831;
  color: white;
  font-size: 50px;
  font-family: "LCD", sans-serif;
  height: 100px;
  width: 80%;
  border: none;
  font-weight: bold;
  text-align: center;
  margin-bottom: 0px;
}

#timeHolder {
  background-color: #123b52;
  color: white;
  display: none;
  margin-bottom: 0px;
}

#speakingTime {
  background-color: #00ffc3;
  outline: none;
  scroll-behavior: smooth;
  height: 50px;
  width: 100px;
  font-weight: bold;
  color: black;
  font-family: Arial, sans-serif;
  font-size: 21px;
  margin: 10px;
  text-align: center;
  border: none;
}

#speakingTime:hover {
  background-color: #c300ff;
}

#status {
  color: white;
}
#pageText {
  color: orange;
}
.titleIsh {
  font-weight: bold;
  font-size: 30px;
}
#volume-visualizer-wrapper {
  background-color: #222831;
  margin-top: 0px;
  padding: 0px;
  margin-bottom: 0px;
  width: 80%;
  display: inline-block;
  padding-bottom: 10px;
}

#volume-visualizer {
  --volume: 0%;
  position: relative;
  height: 10px;
  background-color: #222831;
  margin-top: 0px;
  margin-bottom: 0px;
  width: 100%;
  border: none;
  display: inline-block;
}

#volume-visualizer::before {
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  width: var(--volume);
  background-color: #c300ff;
  transition: width 100ms linear;
}
#container {
  height: 200px;
  margin-bottom: 0px;
  display: none;
  margin-top: -100px;
}

.video-container {
  position: relative;
  margin-top: 0px;
  background-color: #222831;
  width: 80%;
  display: inline-block;
}

canvas {
  position: absolute;
  left: 0;
  top: 0px;
}
.result-container {
  width: 100%;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}
.result-container > div {
  font-size: 1.3rem;
  padding: 0.5rem;
  margin: 5px 0;
  color: white;
  text-transform: capitalize;
}

video {
  width: 100%;
  margin-bottom: -150px;
  margin-top: 0px;
}

#loading {
  height: 50px;
}

#loadingContainer {
  color: #fdfd96;
  margin-bottom: 150%;
  font-size: 50px;
}

#initialMessage {
  font-size: 20px;
  color: #c300ff;
}

#footer {
  display: flex;
  position: float;
  bottom: 0;
  margin-top: 27%;
  height: 3rem;
  text-align: left;
}

#bugs {
  color: white;
  font-size: 12px;
}

@page {
  size: A3 landscape;
  counter-increment: page;
  margin: none;
}
.maindiv {

  height: 100%;
}
.text {
  background: url("https://media.giphy.com/media/l0Exe4tRgXZcvSXDy/giphy.gif");
  background-size: cover;
  -webkit-background-clip: text;
  color: #00000047;
  font-size: 40px;
  font-weight: bold;
}

#mainTitle {
color: hotpink;
font-size: 70px;

}
</style>
