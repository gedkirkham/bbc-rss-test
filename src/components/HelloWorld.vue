<template>
  <div class="hello">
    <table style="width:100%; border:1px dashed green; text-align:left">
      <tr>
        <th v-for="(item, index) in headers" :key="index">{{ item }}</th>
      </tr>
      <tr v-for="(item, rowIndex) in tableData" :key="rowIndex">
        <td v-for="(item, index) in tableData[rowIndex]" :key="index">{{ item }}</td>
      </tr>
    </table>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      tableData: [],
      headers: ['Location'],
      locations: {
        glasgow: '2648579',
        manchester: '2643123',
        london: '2643743'
      }
    }
  },
  async created() {
    let parser = new DOMParser()

    Object.entries(this.locations).forEach(async ([key]) => {
      let fetchedData
      await fetch(`https://weather-broker-cdn.api.bbci.co.uk/en/forecast/rss/3day/${this.locations[key]}`)
        .then(response => response.text())
        .then(text => { fetchedData = text })

      let xmlDoc = parser.parseFromString(fetchedData, "text/xml");
      const itemBlock = xmlDoc.getElementsByTagName("item")
      console.log(itemBlock)
      const title = xmlDoc.getElementsByTagName("title")
      
      let obj = {}
      Object.entries(itemBlock).forEach(item => {
        console.log(item)
        let nodes = item[1].childNodes
        nodes.forEach((item, index) => { if (nodes[index].nodeName !== '#text') obj[nodes[index].nodeName] = nodes[index].innerHTML })
        
        let tableObj = { Location: title[0].innerHTML}
        const descArray = obj.description.split(',')
        descArray.forEach((item, index) => {
          if (!item.includes(':')) {
            let split = descArray[index - 1].split(':')
            tableObj[split[0]] = tableObj[split[0]] + item
          } else {
            let split = item.split(':')
            tableObj[split[0]] = split[1]
            if (this.tableData.length === 0) this.headers.push(split[0])
          }
        })
  
        this.tableData.push(tableObj)
      })
    })

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
