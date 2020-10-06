<template>
  <v-app style="background-color: #505259; overflow: hidden">
    <v-dialog v-model="dialog" max-width="60%">
      <v-card class="pa-3">
        <v-btn
          small
          @click="dialog = false"
          icon
          style="position: absolute; right: 0"
          class="mr-2"
          color="red"
          ><v-icon color="red">mdi-close</v-icon></v-btn
        >
        <div class="font">Tabela Final:</div>
        <v-divider class="mb-3"></v-divider>
        <table border="1" class="ml-2">
          <tr>
            <th v-for="i in restrictions.length + coefs.length + 1" :key="i">
              {{ i != restrictions.length + coefs.length + 1 ? `X${i}` : "b" }}
            </th>
          </tr>
          <tr v-for="(linha, i) in table" :key="i">
            <td class="pa-1" v-for="(col, j) in linha" :key="j">
              {{ col.toFixed(2) }}
            </td>
          </tr>
        </table>
        <div class="font mt-2">Resposta Final:</div>
        <v-divider class="mb-2"></v-divider>
        <div class="ml-2" v-for="resp in resposta.x" :key="resp">
          {{ resp }}
        </div>
        <div class="ml-2">{{ resposta.z }}</div>
      </v-card>
    </v-dialog>

    <v-row justify="center">
      <v-col cols="11" md="9">
        <v-card elevation="12">
          <v-toolbar flat color="#29916d" dark>
            <v-toolbar-title>Algoritmo Simplex</v-toolbar-title>
            <v-spacer></v-spacer>
          </v-toolbar>
          <v-tabs vertical color="#29916d" v-model="index">
            <v-tab> <v-icon left>mdi-exponent</v-icon>Equação z </v-tab>
            <v-tab> <v-icon left>mdi-exclamation</v-icon>Restrições </v-tab>

            <v-tab-item class="py-4" style="border-left: 1px solid grey">
              <div class="ml-2">
                <div class="mb-4">
                  <span class="font">Upload de arquivo txt: </span>
                  <input
                    :key="inputKey"
                    type="file"
                    @change="loadTextFromFile"
                  />
                  <v-btn @click="inputKey++" icon small>
                    <v-icon>mdi-close</v-icon>
                  </v-btn>
                </div>
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

                <span class="mx-2">{{ coefs.length }}</span>

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

                <v-tooltip
                  bottom
                  color="red"
                  v-if="warning && coefs.length > 2"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-icon color="red" v-bind="attrs" v-on="on" class="mx-1"
                      >mdi-alert-outline</v-icon
                    >
                  </template>
                  <span
                    >Remover uma variável agora ira apagar tudo realcionado ao
                    x{{ coefs.length }}</span
                  >
                </v-tooltip>
                <v-divider class="mt-2"></v-divider>

                <div class="font my-5">Maximizar:</div>
                <div class="font">
                  z =
                  <span v-for="(coef, i) in coefs" :key="i">
                    {{ i > 0 ? "+" : "" }}
                    <v-text-field
                      dense
                      v-model.number="coefs[i]"
                      type="number"
                      outlined
                      style="
                        width: 80px;
                        display: inline-block;
                        font-size: 20px;
                        text-align: right;
                      "
                    ></v-text-field>
                    {{ i > 0 ? " x" : "x" }}{{ i + 1 }}
                  </span>
                </div>
              </div>
            </v-tab-item>

            <v-tab-item class="py-4" style="border-left: 1px solid grey">
              <div class="ml-2 font">
                Sujeito a:
                <div
                  class="font my-3"
                  v-for="(rest, n) in restrictions"
                  :key="n"
                >
                  {{ n + 1 }}.)
                  <span v-for="i in coefs.length" :key="i">
                    {{ i > 1 ? "+" : "" }}
                    <v-text-field
                      dense
                      v-model.number="rest.coefs[i - 1]"
                      type="number"
                      outlined
                      style="
                        width: 80px;
                        display: inline-block;
                        font-size: 20px;
                        text-align: right;
                      "
                    ></v-text-field>
                    {{ i > 1 ? " x" : "x" }}{{ i }}
                  </span>

                  <span>
                    ≤
                    <v-text-field
                      dense
                      v-model.number="rest.b"
                      type="number"
                      outlined
                      style="
                        width: 80px;
                        display: inline-block;
                        font-size: 20px;
                        text-align: right;
                      "
                    ></v-text-field>
                  </span>

                  <v-btn
                    icon
                    class="ml-2"
                    @click="removeRestriction(n)"
                    :disabled="restrictions.length == 1 ? true : false"
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
                <v-icon left>mdi-plus</v-icon>Add Restrição
              </v-btn>
            </v-tab-item>
          </v-tabs>
        </v-card>
      </v-col>
    </v-row>

    <v-tooltip left color>
      <template v-slot:activator="{ on, attrs }">
        <v-btn
          large
          fab
          fixed
          color="#d99734"
          bottom
          right
          v-bind="attrs"
          v-on="on"
          @click="generateTable()"
        >
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
    coefs: ["", ""], //vetor de coeficientes da funcao z
    restrictions: [{ coefs: ["", ""], b: "" }], //vetor de restricoes
    index: 0, //indice das tabs
    table: [], //vetor tabela -> representa a tabela como uma matriz bidimensional
    resposta: {
      x: [], //vetor com os valores de Xi (onde i = indice) e valor maximo de Z
      z: null,
    },
    dialog: false, //variavel responsavel por mostrar/esconder dialogo com resposta
    inputKey: 0,
  }),
  methods: {
    reset() {
      //reseta coeficientes e restricoes
      this.coefs = ["", ""];
      this.restrictions = [{ coefs: ["", ""], b: "" }];
      this.index = 0;
    },
    loadTextFromFile(ev) {
      const file = ev.target.files[0];
      const reader = new FileReader();

      this.reset();
      reader.onload = (e) => this.handleText(e.target.result);

      reader.readAsText(file);
    },
    handleText(text) {
      //funcao responsavel por manejar o texto recebido
      let lines = text.split("\n"); //vetor com o texto de cada uma das linhas

      lines.forEach((line, i) => {
        let array = line.replace(/(\r\n|\n|\r)/gm, "").split(" "); //vetor com cada um dos valores da linha em questao
        //o replace foi necessario para que nao houvessem problemas com quebras de linha ou outros tipos de caracteres de formatacao

        if (i == this.restrictions.length + 1) this.addRestriction(); // adiciona uma nova restriçao quando necessario (-1 para desconsiderar a primeira linha que pertence aos coeficientes)

        let cont = 0; //contador de valores validos (nao espacos em branco)
        if (i == 0) {
          //se for a primeira linha -> colocar em coeficientes
          array.forEach((value) => {
            //para cada um dos valores colocar em vetor coefs
            if (value.length) {
              if (cont == this.coefs.length) {
                this.addCoef();
              }
              this.coefs[cont++] = parseFloat(value);
            }
          });
        } else {
          //se nao -> colocar em restricoes
          let aux = 0;
          cont = 0;
          array.forEach((value, n) => {
            //para cada um dos valores
            if (value.length) {
              aux = n; //guarda a posicao do ultimo valor que nao era espaco numerico
              if (n == array.length - 1)
                //o ultimo é sempre o valor de b
                this.restrictions[i - 1].b = parseFloat(value);
              else {
                this.restrictions[i - 1].coefs[cont] = parseFloat(value);
              }

              cont++;
            }
          });

          if (!this.restrictions[i - 1].b.length) {
            //caso tenham sido deixado espacos em branco no final o valor de b nao tera sido atribuido, é, portanto, necessario colocar o valor da ultima posicao de array que possuia um valor que nao fosse espaco em branco
            this.restrictions[i - 1].b = parseFloat(array[aux]);
          }
        }
      });
    },
    addCoef() {
      //funcao repsonsavel por adicionar um lugar para coeficiente
      this.coefs.push(""); //adiciona um novo espaco para coeficiente no vetor de coeficientes

      this.restrictions.forEach((rest) => {
        //adiciona um novo espaco para coeficiente em cada uma das restricoes ja existentes
        rest.coefs.push("");
      });
    },
    addRestriction() {
      //funcao responsavel por adicionar restricoes
      let obj = {
        //objeto auxiliar
        coefs: [],
        b: "",
      };

      this.coefs.forEach(() => obj.coefs.push("")); //adiciona um coeficiente no objeto auxiliar para cada um ja existente no vetor coefs

      this.restrictions.push(obj); //adiciona o objeto no vetor de restricoes
    },
    removeCoef() {
      //funcao responsavel por remover o ultimo coeficiente
      let ok = true;
      if (this.warning) {
        ok = confirm(
          `Você tem certeza que deseja remover tudo relacionado à variável X${this.coefs.length}?`
        );
      }

      if (ok) {
        //se usuario confirmar
        this.coefs.pop(); //remove o ultimo coeficiente do vetor de coeficientes

        this.restrictions.forEach((rest) => {
          //remove o ultimo coeficiente de cada uma das restricoes ja existentes
          rest.coefs.pop();
        });
      }
    },
    removeRestriction(index) {
      this.restrictions.splice(index, 1);
    },
    generateTable() {
      let nFolgas = this.restrictions.length; //uma variavel de folga pra cada restricao
      this.table = [];

      for (let i = 0; i <= this.restrictions.length; i++) {
        //numero de linhas vai ser igual ao numero de restricoes + 1 para a linha dos coefs da funcao
        let aux = [];
        for (let j = 0; j < nFolgas + this.coefs.length + 1; j++) {
          //numero de colunas é igual ao numero de folgas somado com o numero de coeficientes +1 para os b's
          if (i == this.restrictions.length) {
            //esta na linha dos coeficientes
            if (j < this.coefs.length) aux.push(-this.coefs[j]);
            //coloca coeficientes ate a ultima coluna de coeficientes multiplicados por -1 (simluando igualar a equacao a 0)
            else aux.push(0); //se a coluna nao for de coeficiente colocar 0
          } else {
            if (j < this.coefs.length) {
              //enquanto coluna dentro das colunas de coeficientes adicionar do vetor coefs da restricao
              aux.push(this.restrictions[i].coefs[j]);
            } else if (j < nFolgas + this.coefs.length) {
              //enquanto nas variveis de folga
              if (j - this.coefs.length == i) aux.push(1);
              //se a linha for a mesma da coluna da variavel de folga adiciona 1
              else aux.push(0); //se nao adiciona 0
            } else {
              aux.push(this.restrictions[i].b); //na ultima coluna adiciona o valor de b
            }
          }
        }
        this.table.push(aux); //adiciona linha na tabela
      }
      this.handleTable();
    },
    handleTable() {
      let isOptimal = this.validaLinha() == -1 ? true : false; //inicializa verificando se ha algum valor negativo na ultima linha da tabela -> true se todos os valores na ultima linha forme maiores ou iguais a 0

      while (!isOptimal) {
        //repete ate que todos os valores na linha de coeficientes da tabelas seja maiores ou iguais a 0
        // debugger;
        //pegar o menor valor da linha dos coeficientes
        let linhaCoefs = this.table.length - 1;
        let menor = Math.min(...this.table[linhaCoefs]);

        //definir coluna pivo
        let pivo = this.table[linhaCoefs].indexOf(menor);

        //definir a linha que sai da base
        let colunaB = this.table[0].length - 1; //coluna b é a ultima
        let divs = this.table.map((linha, i) => {
          if (linha[pivo] !== 0 && i !== this.table.length - 1) {
            //evita divisoes por zero e a ultima linha (coefs de z)
            let div = linha[colunaB] / linha[pivo];
            return Object.assign({}, { value: div, line: i }); //retorna objeto com valor e indice da linha
          }
        }); //vetor de divisoes da coluna b pela coluna pivo em cada linha

        let menorDiv = { value: null };
        divs.forEach((div) => {
          if (div !== undefined) {
            if (div.value >= 0) {
              if (menorDiv.value == null) {
                //primeira divisao valida
                menorDiv = Object.assign({}, div);
              } else if (div.value < menorDiv.value)
                menorDiv = Object.assign({}, div); //guarda se divisor for menor que o salvo ate o momento
            }
          }
        });

        let linhaExit = menorDiv.line; //guarda a linha do menor divisor

        //dividir linha pelo numero em linhaExit*pivo -> precisa ser = 1
        let divisor = this.table[linhaExit][pivo];
        this.table[linhaExit].forEach((col, j) => {
          this.table[linhaExit][j] /= divisor;
        });

        //zerar toda a coluna pivo exceto a linhaExit
        let divisao = null;
        this.table.forEach((linha, i) => {
          if (i != linhaExit) {
            divisao = this.table[i][pivo] / this.table[linhaExit][pivo];
            linha.forEach((col, j) => {
              this.table[i][j] -= divisao * this.table[linhaExit][j];
            });
          }
        });

        pivo = this.validaLinha();
        isOptimal = pivo == -1 ? true : false;
      }
      this.handleResposta();
    },
    validaLinha() {
      //funcao responsavel por verificar se solucao é otima de acordo com as colunas da ultima linha
      let linha = this.table.length - 1; //pega indice da ultima linha da tabela
      let menor = Math.min(...this.table[linha]);

      //definir coluna pivo
      if (menor >= 0) return -1; ////se o menor valor encontrado nao for negativo retorna -1 -> indica que nao há numeros negativos na ultima linha (solucao otima)

      return this.table[linha].indexOf(menor);

      // let x = this.table[linha].find((col) => col < 0); //procura por um valor negativo na linha
      // if (x == undefined) return -1;

      // return this.table[linha].indexOf(x); //se foi encontrado um valor negativo retor a coluna dele sendo necessario ainda fazer operacoes na tabela
    },
    handleResposta() {
      this.resposta.x = []; //reiniciando o vetor de x em resposta

      let colunaB = this.table[0].length - 1;
      for (let j = 0; j < this.table[0].length; j++) {
        let found = false;
        for (let i = 0; i < this.table.length - 1; i++) {
          if (j != colunaB) {
            let linhaResp = this.validaCol(j);
            if (linhaResp != -1 && !found) {
              //se nao for -1 é porque essa variavel tem um resultado
              found = true;
              let value = this.table[linhaResp][colunaB]; //valor é igual ao valor da ultima coluna (coluna b) da tabela
              this.resposta.x.push(
                `X${j + 1} = ${parseFloat(value).toFixed(2)}`
              );
            }
          } else if (i + 1 == this.table.length - 1) {
            this.resposta.z = `Z = ${parseFloat(this.table[i + 1][j]).toFixed(
              2
            )}`;
          }
        }
      }
      this.dialog = true;
    },
    validaCol(col) {
      let valid = true;
      let nOnes = 0;
      let linhaDeOne = null;
      this.table.forEach((linha, i) => {
        if (linha[col] != 1 && linha[col] != 0) valid = false;
        if (linha[col] == 1) {
          nOnes++;
          linhaDeOne = i;
        }
      });

      if (valid == true && nOnes == 1) return linhaDeOne;

      return -1;
    },
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
</style>