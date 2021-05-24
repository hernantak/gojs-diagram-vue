<template>
  <b-container >
      <div id="app">
        <h3>Diagram Hubungan </h3>
          <div id="myDiagramDiv"></div>
            <b-row cols="2" class="mb-4">
              <p>Nama</p>
              <input v-model="data_node" placeholder="Nama">
              <p>Kenalan</p>
              <b-form-select
                name="kenalan"
                v-model="data_link"
                :options="diagramData.nodeDataArray">
              </b-form-select>
              <p>Hubungan</p>
              <input v-model="data_hubungan" placeholder="Hubungan"><br><br>
            </b-row>
            <b-button v-on:click="addNode" variant="success" class="mb-4">Tambah</b-button>
            <div class="text-align: left;">
              <p>Info: Untuk hapus link atau node cukup klik dan tekan "Delete" pada keyboard</p>
            </div>
      </div>
  </b-container>
</template>

<script>
import go from 'gojs'
import dataJson from './data/data.json'
import linkDataJson from './data/linkdata.json'

export default {
  name: "App",
  data () {
    return {
      diagramz: '',
      check: '',
      parent: '',
      data_node: '',
      data_link: '',
      data_hubungan: '',
      counter: 3,
      diagramData: { 
        nodeDataArray: dataJson,
        linkDataArray: linkDataJson
      }
    }
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
    filterData() {
      // check for node & link
      let data = this.diagramData.nodeDataArray;
      for(let i of data) {
        if(this.data_node.toLowerCase() === i.text.toLowerCase()) {
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
        this.diagramz.model.addLinkData({ from: this.data_link, to: counter, text: this.data_hubungan });
        this.diagramz.model.commitTransaction("added Node and Link");
        this.check = '';
      } else {
        this.diagramz.model.startTransaction();
        this.diagramz.model.addLinkData({ from: this.parent.key, to: this.data_link, text: this.data_hubungan});
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
</style>
