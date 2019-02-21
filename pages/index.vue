<template>
  <div>
    <table>
      <tr>
        <td class="col1">
          Data:
          <textarea v-model="data" />
          Schema:
          <textarea v-model="schema" />
          <button class="submit" @click="notarize">Notarize</button>
        </td>
        <td class="col2">
          Output:
          <pre class="output">{{ output }}</pre>
          Imprint
          <pre class="imprint">{{ imprint }}</pre>
        </td>
      </tr>
    </table>
  </div>
</template>

<script>
import { Cert } from '@0xcert/cert';

export default {
  data() {
    return {
      data: "",
      schema: "",
      output: "",
      imprint: "",
    }
  },
  methods: {
    async notarize() {
      const cert = new Cert({ schema: JSON.parse(this.schema) })
      const recipes = await cert.notarize(JSON.parse(this.data))

      const keys = {}
      recipes.forEach((recipe) => {
        keys[recipe.nodes[0].hash] = recipe.path.join('.')
      })

      this.output = recipes.map((recipe) => {
        return [
          recipe.path.join('.'),
          ': ',
          ...recipe.values.map((v) => {
            if (v.value.length == 64) {
              return keys[v.value]
            } else {
              return v.value
            }
          }).join(', ')
        ].join('');
      }).join('\n')

      this.imprint = await cert.imprint(recipes)
    }


  }
}
</script>

<style>
table {
  width: 100%;
}
.col1, .col2 { 
  width: 50%;
  padding: 20px;
  vertical-align: top;
}
textarea {
  width: 100%;
  height: 300px;
}
.output {
  height: 496px;
  background: #c0c0c0;
}
.imprint {
  height: 80px;
  background: #c0c0c0;
}
.submit {
  width: 200px;
  background: #ff0000;
  line-height: 30px;
  color: #ffffff;
}
</style>
