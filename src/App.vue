<template>
  <v-app style="background-color: #505259; overflow:hidden">
    <input type="file" @change="loadTextFromFile">
    <v-row justify="center">
      <v-col cols="11" md="9">
        <v-card elevation="12">
          <v-toolbar flat color="#29916d" dark>
            <v-toolbar-title>Algoritmo Simplex</v-toolbar-title>
          </v-toolbar>
          <v-tabs vertical color="#29916d" v-model="index">
            <v-tab>
              <v-icon left>mdi-exponent</v-icon>Equação z
            </v-tab>
            <v-tab>
              <v-icon left>mdi-exclamation</v-icon>Restrições
            </v-tab>

            <v-tab-item class="py-4" style="border-left: 1px solid grey;">
              <div class="ml-2">
                <span class="font">Numero de variáveis:</span>
                <v-tooltip bottom color>
                  <template v-slot:activator="{ on, attrs }">
                    <v-btn
                      icon
                      x-small
                      class="mx-1"
                      @click="removeCoef()"
                      style="border: 1px solid black"
                      v-bind="attrs"
                      v-on="on"
                      :disabled="coefs.length == 2 ? true : false"
                    >
                      <v-icon>mdi-minus</v-icon>
                    </v-btn>
                  </template>
                  <span>Remover</span>
                </v-tooltip>

                <span class="mx-2">{{coefs.length}}</span>

                <v-tooltip bottom color>
                  <template v-slot:activator="{ on, attrs }">
                    <v-btn
                      icon
                      x-small
                      class="mx-1"
                      @click="addCoef()"
                      style="border: 1px solid black"
                      v-bind="attrs"
                      v-on="on"
                    >
                      <v-icon>mdi-plus</v-icon>
                    </v-btn>
                  </template>
                  <span>Adicionar</span>
                </v-tooltip>

                <v-tooltip bottom color="red" v-if="warning">
                  <template v-slot:activator="{ on, attrs }">
                    <v-icon color="red" v-bind="attrs" v-on="on" class="mx-1">mdi-alert-outline</v-icon>
                  </template>
                  <span>Remover uma variável agora ira apagar tudo realcionado ao x{{coefs.length}}</span>
                </v-tooltip>
                <v-divider class="mt-2"></v-divider>

                <div class="font my-5">Maximizar:</div>
                <div class="font">
                  z =
                  <span v-for="(coef, i) in coefs" :key="i">
                    {{i > 0 ? '+' : ''}}
                    <v-text-field
                      dense
                      v-model.number="coefs[i]"
                      type="number"
                      outlined
                      style="width: 80px; display: inline-block; font-size:20px; text-align:right"
                    ></v-text-field>
                    {{i > 0 ? ' x' : 'x'}}{{i + 1}}
                  </span>
                </div>
              </div>
            </v-tab-item>

            <v-tab-item class="py-4" style="border-left: 1px solid grey">
              <div class="ml-2 font">
                Sujeito a:
                <div class="font my-3" v-for="(rest, n) in restrictions" :key="n">
                  {{n + 1}}.)
                  <span v-for="i in coefs.length" :key="i">
                    {{i > 1 ? '+' : ''}}
                    <v-text-field
                      dense
                      v-model.number="rest.coefs[i-1]"
                      type="number"
                      outlined
                      style="width: 80px; display: inline-block; font-size:20px; text-align:right"
                    ></v-text-field>
                    {{i > 1 ? ' x' : 'x'}}{{i}}
                  </span>

                  <span>
                    ≤
                    <v-text-field
                      dense
                      v-model.number="rest.b"
                      type="number"
                      outlined
                      style="width: 80px; display: inline-block; font-size:20px; text-align:right"
                    ></v-text-field>
                  </span>

                  <v-btn
                    icon
                    class="ml-2"
                    @click="removeRestriction(n)"
                    :disabled=" restrictions.length == 1 ? true : false"
                  >
                    <v-icon>mdi-delete</v-icon>
                  </v-btn>
                </div>
              </div>
              <v-btn
                  style="display: block"
                  class="mb-3"
                  text
                  color="primary"
                  @click="addRestriction()"
                >
                <v-icon left>mdi-plus</v-icon>
                 Add Restrição</v-btn>
            </v-tab-item>
          </v-tabs>
        </v-card>
      </v-col>
    </v-row>

    <v-tooltip left color>
      <template v-slot:activator="{ on, attrs }">
        <v-btn large fab fixed color="#d99734" bottom right v-bind="attrs" v-on="on">
          <v-icon color="white" x-large>mdi-calculator</v-icon>
        </v-btn>
      </template>
      <span>Calcular</span>
    </v-tooltip>
  </v-app>
</template>

<script>
export default {
  name: "App",

  data: () => ({
    coefs: ["", ""],
    restrictions: [{ coefs: ["", ""], b: "" }],
    index: 0,
  }),
  methods: {
    addCoef() { //funcao repsonsavel por adicionar um lugar para coeficiente
      this.coefs.push(""); //adiciona "" no vetor de coeficientes

      this.restrictions.forEach((rest) => { //adiciona um novo coeficiente em cada uma das restricoes ja existentes
        rest.coefs.push("");
      });
    },
    addRestriction() { //funcao responsavel por adicionar restricoes
      let obj = { //objeto auxiliar
        coefs: [],
        b: "",
      };

      this.coefs.forEach(() => obj.coefs.push("")) //adiciona um coeficiente no objeto auxiliar para cada um ja existente no vetor coefs

      this.restrictions.push(obj); //adiciona o objeto no vetor de restricoes
    },
    removeCoef() { //funcao responsavel por remover o ultimo coeficiente
      this.coefs.pop(); //remove o ultimo coeficiente do vetor de coeficientes

      this.restrictions.forEach((rest) => { //remove o ultimo coeficiente de cada uma das restricoes ja existentes
        rest.coefs.pop();
      });
    },
    removeRestriction(index) {
      console.log(index);
      this.restrictions.splice(index, 1);
    },
    generateTable() {

    },
    reset() { //reseta coeficientes e restricoes
      this.coefs = ["", ""];
      this.restrictions = [{ coefs: ["", ""], b: "" }];
      this.index = 0;
    },
    loadTextFromFile(ev) {
      const file = ev.target.files[0];
      const reader = new FileReader();

      this.reset();
      reader.onload = e => this.handleText(e.target.result);
      console.log("here");
      reader.readAsText(file);
    },
    handleText(text) { //funcao responsavel por manejar o texto recebido
      let lines = text.split("\n"); //vetor com o texto de cada uma das linhas

      lines.forEach((line, i) => {
        let array = line.replace(/(\r\n|\n|\r)/gm, "").split(" "); //vetor com cada um dos valores da linha em questao
        //o replace foi necessario para que nao houvessem problemas com quebras de linha ou outros tipos de caracteres de formatacao

        if(i == this.restrictions.length + 1) this.addRestriction(); // adiciona uma nova restriçao quando necessario (-1 para desconsiderar a primeira linha que pertence aos coeficientes)

        if(i == 0) { //se for a primeira linha -> colocar em coeficientes
          array.forEach((value, n) => { //para cada um dos valores colocar em vetor coefs
            if(n == this.coefs.length) this.addCoef();
            this.coefs[n] = value;
          })
        } else { //se nao -> colocar em restricoes
          array.forEach((value, n) => { //para cada um dos valores 
            if(n == array.length - 1)//o ultimo é sempre o valor de b
              this.restrictions[i-1].b = value;
            else {
              this.restrictions[i-1].coefs[n] = value
            }
          })
        }
      })
    }
  },
  computed: {
    warning() {
      if (this.coefs[this.coefs.length - 1] !== "") return true;

      for (let i = 0; i < this.restrictions.length; i++)
        if (
          this.restrictions[i].coefs[this.restrictions[i].coefs.length - 1] !==
          ""
        )
          return true;

      //forEach nao funciona em computed??
      // this.restrictions.forEach(rest => {
      //   console.log(rest.coefs[rest.coefs.length-1])
      //   if(rest.coefs[rest.coefs.length-1] !== '') return true;
      // })

      return false;
    },
  },
};
</script>

<style scoped>
.font {
  font-size: 22px;
  font-weight: 500;
  color: rgb(61, 61, 61);
}
.card {
  max-height: 400px;
  overflow: auto;
}
.card::-webkit-scrollbar {
  width: 4px;
}
/* Track */
.card::-webkit-scrollbar-track {
  background: none;
}
/* Handle */
.card::-webkit-scrollbar-thumb {
  background: #f1ac4a;
  border-radius: 5px;
}
/* Handle on hover */
.card::-webkit-scrollbar-thumb:hover {
  background: #fca326;
}
</style>
