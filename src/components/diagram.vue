<template>

  <div id="app">
    <div class="vx-row">
      <div id="myDiagramDiv"></div>
      <div class="vx-row">
        <div class="vx-col w-full">
          <p>Nama</p>
          <input v-model="data_node" placeholder="Nama">
        </div>
      </div>
        <p>Kenalan</p>
        <b-form-select
          name="kenalan"
          v-model="select_kenalan"
          :options="diagramData.nodeDataArray"></b-form-select>
        <p>Hubungan</p>
        <input v-model="data_hubungan" placeholder="Hubungan"><br><br>
        <button v-on:click="addNode">Add</button>

        {{ select_kenalan }}
    </div>
  </div>
</template>

<script>
import go from 'gojs'
// import vSelect from 'vue-select'
import dataJson from './data/data.json'
import linkDataJson from './data/linkdata.json'

export default {
  name: "App",
  data () {
    return {
      select_kenalan: '',
      diagramz: '',
      check: '',
      parent: '',
      data_node: '',
      data_link: '',
      data_hubungan: '',
      counter: 4,
      diagramData: { 
        nodeDataArray: dataJson,
        linkDataArray: linkDataJson
      },
    }
  },
  components: {
    // 'v-select': vSelect
  },
  methods : {
    init () {
      var $ = go.GraphObject.make;  // for conciseness in defining templates
      var myDiagram =
        $(go.Diagram, "myDiagramDiv",  // create a Diagram for the DIV HTML element
          { // enable undo & redo
            "undoManager.isEnabled": true,
            initialAutoScale: go.Diagram.Uniform,  // an initial automatic zoom-to-fit
            layout:
            $(go.ForceDirectedLayout,  // automatically spread nodes apart
              { maxIterations: 200, defaultSpringLength: 30, defaultElectricalCharge: 100 })
          });

      // define a simple Node template
      myDiagram.nodeTemplate =
        $(go.Node, "Auto",  // the whole node panel
          { locationSpot: go.Spot.Center },
          // define the node's outer shape, which will surround the TextBlock
          $(go.Shape, "RoundedRectangle",
            { fill: $(go.Brush, "Linear", { 0: "rgb(254, 201, 0)", 1: "rgb(254, 162, 0)" }), stroke: "black" }),
          $(go.TextBlock,
            { font: "bold 10pt helvetica, bold arial, sans-serif", margin: 4 },
            new go.Binding("text", "text")),

            // {
            //   contextMenu:     // define a context menu for each node
            //     $("ContextMenu",  // that has one button
            //       $("ContextMenuButton",
            //         {
            //           "ButtonBorder.fill": "white",
            //           "_buttonFillOver": "skyblue"
            //         },
            //         $(go.TextBlock, "Change Color"),
            //         { click: this.changeColor })
            //       // more ContextMenuButtons would go here
            //     )  // end Adornment
            // }

        );

      myDiagram.linkTemplate =
        $(go.Link,  // the whole link panel
          $(go.Shape,  // the link shape
            { stroke: "black" }),
          $(go.Shape,  // the arrowhead
            { toArrow: "standard", stroke: null }),
          $(go.Panel, "Auto",
            $(go.Shape,  // the label background, which becomes transparent around the edges
              {
                fill: $(go.Brush, "Radial", { 0: "rgb(240, 240, 240)", 0.3: "rgb(240, 240, 240)", 1: "rgba(240, 240, 240, 0)" }),
                stroke: null
              }),
            $(go.TextBlock,  // the label text
              {
                textAlign: "center",
                font: "10pt helvetica, arial, sans-serif",
                stroke: "#555555",
                margin: 4
              },
              new go.Binding("text", "text"))
          )
        );        

      myDiagram.model = new go.GraphLinksModel(this.diagramData.nodeDataArray, this.diagramData.linkDataArray)
      this.diagramz = myDiagram;
    },

    // changeColor: function() {
      // this.diagramz.commit(function(d) {
      //   // get the context menu that holds the button that was clicked
      //   var contextmenu = obj.part;
      //   // get the node data to which the Node is data bound
      //   var nodedata = contextmenu.data;
      //   // compute the next color for the node
      //   var newcolor = "lightblue";
      //   switch (nodedata.color) {
      //     case "lightblue": newcolor = "lightgreen"; break;
      //     case "lightgreen": newcolor = "lightyellow"; break;
      //     case "lightyellow": newcolor = "orange"; break;
      //     case "orange": newcolor = "lightblue"; break;
      //   }
      //   // modify the node data
      //   // this evaluates data Bindings and records changes in the UndoManager
      //   d.model.set(nodedata, "color", newcolor);
      // }, "changed color");
    // },    

    filterData() {
      // check for node & link
      let data = this.diagramData.nodeDataArray;
      for(let i of data) {
        if(this.data_node === i.text) {
          this.parent = i;
          this.check = i.key;
        } 
      }
    },

    addNode: function() {
      this.filterData();
      if(!this.check) {
        var counter = ++this.counter;
        this.diagramz.model.startTransaction();
        this.diagramz.model.addNodeData({ value: counter, key: counter, text: this.data_node });
        this.diagramz.model.addLinkData({ from: this.select_kenalan, to: counter, text: this.data_hubungan });
        this.diagramz.model.commitTransaction("added Node and Link");
        this.check = '';
      } else {
        this.diagramz.model.startTransaction();
        this.diagramz.model.addLinkData({ from: this.parent.key, to: this.select_kenalan, text: this.data_hubungan});
        this.diagramz.model.commitTransaction("added Node and Link");
        this.check = '';
      }
    },
  },
  mounted() {
    this.init()
  }
};
</script>

<style>
#myDiagramDiv {
  width: 100%;
  height: 500px;
}

.style-chooser .vs__search::placeholder,
.style-chooser .vs__dropdown-toggle,
.style-chooser .vs__dropdown-menu {
  background: #dfe5fb;
  border: none;
  color: #394066;
  text-transform: lowercase;
  font-variant: small-caps;
}

.style-chooser .vs__clear,
.style-chooser .vs__open-indicator {
  fill: #394066;
}
</style>
