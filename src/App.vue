<!-- Inspiration from https://adjacency-matrix-diagram.vercel.app/ -->
<template>
  <img alt="CT logo" src="https://cushingterrell.com/wp-content/uploads/2020/01/CT_Logo_R_105x50.fw_.png">
  <div id="app-inner">
    <h3>Adjacency Diagram Editor</h3>

    <div id="editor-wrap">
        <div id="editor-outer" class="row">
            <div id="editor" class="half">
                <textarea cols="15" rows="8" v-model="rawRooms"></textarea>
                <div>
                    <button @click.prevent="loadSamples">Need a sample?</button>
                </div>
                
                <div>
                    <button v-if="rooms.length > 0" @click.prevent="downloadSvg">Download SVG image</button>
                    <button v-if="rooms.length > 0" @click.prevent="downloadState">Download current state</button>
                </div>
                
                <div>
                    <label>Load saved state <input type="file" id="file" @change="handleStateFile"></label>
                </div>
            </div>

            <div id="diagram" class="half">
                <svg width="500" height="500" id="svg-image">
                    <g v-for="(r, ri) in modelRooms" :key="'room-'+ri" :transform="r.g.transform" 
                        class="room-wrap"
                        :class="{active: isActive({ r, ri })}"
                    >
                        <line :x1="r.l1.x1" :y1="r.l1.y1" :x2="r.l1.x2" :y2="r.l1.y2" class="line"/>
                        <text :y="height / 2 + 5">
                            <tspan>{{ r.name }}</tspan>
                        </text>
                        <line v-if="r.l2" :x1="r.l2.x1" :y1="r.l2.y1" :x2="r.l2.x2" :y2="r.l2.y2" class="line"/>
                    </g>

                    <g v-for="(r, ri) in modelRects" :key="`cell-${ri}`" :transform="r.g.transform" 
                        class="rect-wrap"
                        @mouseover="handleMouseOverRect({ r, ri })"
                        @mouseout="handleMouseOutRect({ r, ri })"                    
                    >
                      <rect :x="r.rect.x" :y="r.rect.y" :width="r.rect.width" :height="r.rect.height" :state="r.state"
                        @click="changeState({ r, ri })"
                        class="rect"
                      ></rect>
                      <StateProxEssential v-if="r.state === 'ProxE'" @click="changeState({ r, ri })"/>
                      <StateProxDesire v-if="r.state === 'ProxD'" @click="changeState({ r, ri })"/>
                      <StateSepDesire v-if="r.state === 'SepD'" @click="changeState({ r, ri })"/>
                      <StateSepEssential v-if="r.state === 'SepE'" @click="changeState({ r, ri })"/>
                    </g>        

                    <g :transform="`translate(${length * baseLengthUnit + rooms.length * baseLengthUnit + 3 * baseLengthUnit + 50}, 0)`">
                      <g transform="translate(0,0)">
                        <StateProxEssential/>
                        <text :x="30" :y="15">
                          <tspan>Proximity Essential</tspan>
                        </text>
                      </g>
                      <g transform="translate(0,30)">
                        <StateProxDesire/>
                        <text :x="30" :y="15">
                          <tspan>Proximity Desired</tspan>
                        </text>
                      </g>
                      <g transform="translate(0,60)">
                        <text :x="30" :y="15">
                          <tspan>No Spatial Relation</tspan>
                        </text>
                        
                      </g>
                      <g transform="translate(0,90)">
                        <text :x="30" :y="15">
                          <tspan>Separation Desired</tspan>
                        </text>
                        <StateSepDesire/>
                      </g>
                      <g transform="translate(0,120)">
                        <text :x="30" :y="15">
                          <tspan>Separation Essential</tspan>
                        </text>
                        <StateSepEssential/>
                      </g>
                    </g>     
                </svg>
            </div>
        </div>
    </div>

  </div>
</template>

<script>
//import HelloWorld from './components/HelloWorld.vue'  

//    name: 'App',
    import StateProxEssential from './components/StateProxEssential.vue';
    import StateProxDesire from './components/StateProxDesire.vue';
    import StateSepDesire from './components/StateSepDesire.vue';
    import StateSepEssential from './components/StateSepEssential.vue';

    export default {
        components: {
            StateProxEssential,
            StateProxDesire,
            StateSepDesire,
            StateSepEssential
        },
        data() {
            return {
                allStates: ['ProxE', 'ProxD', 'SepD', 'SepE','--'],
                height: 30,
                baseLengthUnit: 10,
                rooms: [],
                states: {},
                currentRooms: [],
                rawRooms: ''
            }
        },
        computed: {
            length() {
                if (this.rooms.length <= 0) return 0;
                let rooms = this.rooms
                    .slice()
                    .sort(((a, b) => b.name.length - a.name.length));
                return rooms[0].name.length;
            },
            modelRooms() {
                return this.rooms.map((r, ri) => {
                    let isLast = ri === this.rooms.length - 1;
                    r.g = {
                        transform: `translate(0,${this.height * ri})`
                    }
                    r.l1 = {
                        x1: 0,
                        y1: 0,
                        x2: this.length * this.baseLengthUnit,
                        y2: 0
                    };
                    if (isLast) {
                        r.l2 = {
                            x1: 0,
                            y1: this.height,
                            x2: this.length * this.baseLengthUnit,
                            y2: this.height
                        };
                    }
                    return r;
                });
            },
            modelRects() {
                let numLayers = this.rooms.length - 1;
                let rectHeight = this.height / Math.SQRT2;
                let rects = [];
                for (let _i=0; _i<numLayers; _i++) {
                    for (let _j=0; _j<numLayers-_i; _j++) {
                        let gx = (this.length * this.baseLengthUnit) + (this.height / 2) + (_j * this.height/2);
                        let gy = _i * this.height + (this.height / 2) + (_j * this.height/2);
                        let rect = {
                            from: this.rooms[_i],
                            to: this.rooms[_j+_i+1],
                            g: { transform: `translate(${gx},${gy}) rotate(45 0 0)`},
                            rect: {
                                x: 0,
                                y: 0,
                                width: rectHeight,
                                height: rectHeight,
                            }
                        };
                        let stateKey = `${rect.from.name}-${rect.to.name}`;
                        if (this.states[stateKey]) {
                            rect.state = this.states[stateKey];
                        }
                        rects.push(rect);
                    }
                }
                return rects;
            },
            canStore() {
                return storageAvailable('localStorage');
            },
            appState() {
                return {
                    states: this.states,
                    rawRooms: this.rawRooms,
                }
            }
        },
        created() {
            this.loadState();
        },
        watch: {
            rawRooms(val) {
                if (val) {
                    this.rooms = val
                        .trim()
                        .split('\n')
                        .filter((v, i, a) => a.indexOf(v) === i)
                        .map(txt => {
                            return {
                                name: txt.trim()
                            }
                        });
                    this.saveState();
                } else {
                    this.rooms = [];
                }
            }
        },
        methods: {
            loadSamples() {
                this.rawRooms = `Entry Foyer\nDinning Room\nMaster Bedroom\nLiving Room\nMaster Bathroom`;
                this.states = {};
            },
            changeState({ r }) {
                let key = `${r.from.name}-${r.to.name}`;
                console.log(key)
                console.log('initial state: ' + this.states[key])
                let state = this.allStates[0];
                if (this.states[key]) {
                    let curStateIndex = this.allStates.indexOf(this.states[key]);
                    let nextStateIndex = curStateIndex < this.allStates.length - 1 ? curStateIndex + 1 : 0;
                    state = this.allStates[nextStateIndex];
                }
                this.states[key] = state;
                console.log('final state:' + state)
                console.log('****')
                this.saveState();
            },
            handleMouseOverRect({ r }) {
                this.currentRooms = [r.from.name, r.to.name];
            },
            isActive({ r }) {
                return this.currentRooms && this.currentRooms.includes(r.name);
            },
            handleMouseOutRect() {
                this.currentRooms = [];
            },
            saveState() {
                if (this.canStore) {
                    window.localStorage.setItem('appState', JSON.stringify(this.appState));
                }
            },
            loadState(rawState) {
                if (!rawState && this.canStore) {
                    rawState = window.localStorage.getItem('appState');
                } 
                if (rawState) {
                    try {
                        let savedState = JSON.parse(rawState);
                        this.rawRooms = savedState.rawRooms;
                        Object.keys(savedState.states).forEach((k) => {
                            this.states[k] = savedState.states[k];
                        })
                    } catch(err) {
                        alert("Cannot load saved state");
                    }
                }
            },
            createDownloadLink(data, fileName) {
                let linkEl = document.createElement('a');
                linkEl.setAttribute("href", data);
                linkEl.setAttribute("download", fileName);
                return linkEl;
            },
            downloadState() {
                let stateTxt = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(this.appState));
                let linkEl = this.createDownloadLink(stateTxt, "adjacency-matrix-diagram.json");
                linkEl.click();
                linkEl.remove();
            },
            handleStateFile(evt) {
                let input = evt.target;
                if (input.files.length === 0) {
                    return alert("Error! You must select adjacency-matrix-diagram JSON state file.");
                }
                if (!input.files[0].type.match(/json/)) {
                    return alert("Error! It's not JSON file.");
                }
                let vm = this;
                let reader = new FileReader();
                reader.onload = function() {
                  let text = reader.result;
                  vm.loadState(text)
                  alert("Success! State loaded.");
                  input.value = "";
                };
                reader.readAsText(input.files[0]);
            },
            downloadSvg() {
                let svg = document.getElementById("svg-image");
                let serializer = new XMLSerializer();
                let source = serializer.serializeToString(svg);
                if(!source.match(/^<svg[^>]+xmlns="http:\/\/www\.w3\.org\/2000\/svg"/)){
                    source = source.replace(/^<svg/, '<svg xmlns="http://www.w3.org/2000/svg"');
                }
                if(!source.match(/^<svg[^>]+"http:\/\/www\.w3\.org\/1999\/xlink"/)){
                    source = source.replace(/^<svg/, '<svg xmlns:xlink="http://www.w3.org/1999/xlink"');
                }
                // add css - needs updated
                source = source.replace(/id="svg-image">/, 'id="svg-image"><style>circle.state-prox-e{fill:#000;stroke:#000}circle.state-prox-d{fill:#fff;stroke:#000}line{fill:#fff;stroke:#000}.room-wrap text{fill:#cacaca}.line{stroke:#000;stroke-width:1px}.triangle{stroke:#000;fill:#fff}.rect{fill:#fff;stroke:#000;cursor:pointer}.active text{fill:#000}.row{display:-webkit-box;display:-ms-flexbox;display:flex}.half{width:50%;padding:5px}#editor textarea{width:100%}svg{width:100%}</style>')
                source = '<?xml version="1.0" standalone="no"?>\r\n' + source;
                source =  'data:text/xml;charset=utf-8,'+encodeURIComponent(source);
                let linkEl = this.createDownloadLink(source, "adjacency-matrix-diagram.svg"); 
                linkEl.click();
                linkEl.remove();
            }
        }
    }

    function storageAvailable(type) {
        try {
            var storage = window[type],
                x = '__storage_test__';
            storage.setItem(x, x);
            storage.removeItem(x);
            return true;
        }
        catch(e) {
            return e instanceof DOMException && (
                // everything except Firefox
                e.code === 22 ||
                // Firefox
                e.code === 1014 ||
                // test name field too, because code might not be present
                // everything except Firefox
                e.name === 'QuotaExceededError' ||
                // Firefox
                e.name === 'NS_ERROR_DOM_QUOTA_REACHED') &&
                // acknowledge QuotaExceededError only if there's something already stored
                storage.length !== 0;
        }
    }  
</script>

<style>
  .room-wrap text {
      fill: #cacaca;   
  }
  .line {
      stroke: #000;
      stroke-width: 1px;
  }
  .triangle {
      stroke: black;
      fill: white;
  }
  .rect {
      fill: #fff;
      stroke: black;
      cursor: pointer;
  }
  .active text {
      fill: #000;
  }
  .row {
      display: flex;
  }
  .half {
      width: 50%;
      padding: 5px;
  }
  #editor textarea {
      width: 100%;
  }
  svg {
      width: 100%;
  }
</style>
