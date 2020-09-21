<template>
  <v-app style="background-color: #505259; overflow:hidden">
    <input type="file" @change="loadTextFromFile" />
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
      this.coefs.pop(); //remove o ultimo coeficiente do vetor de coeficientes

      this.restrictions.forEach((rest) => {
        //remove o ultimo coeficiente de cada uma das restricoes ja existentes
        rest.coefs.pop();
      });
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
          //numero de colunas é igual ao numero de coeficientes somado com o numero de coeficientes +1 para os b's
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
              if (j - 2 == i) aux.push(1);
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

        //pegar o menor valor da linha dos coeficientes
        let linhaCoefs = this.table.length - 1;
        let menor = Math.min(...this.table[linhaCoefs]);

        //definir coluna pivo
        let pivo = this.table[linhaCoefs].indexOf(menor);

        //definir a linha que sai da base
        let colunaB = this.table[0].length - 1; //coluna b é a ultima
        let linhaExit = 0; //inicializa a linha que sai em 0
        let menorDiv = this.table[0][colunaB] / this.table[0][pivo]; //inicializa menorDiv com a primeira divisao

        this.table.forEach((linha, i) => {
          if (i < this.table.length - 1 && linha[pivo] !== 0) {
            //exclui a ultima linha -> linha de Z e ignora linhas com 0 (pra nao ocorrer divisao por 0)
            let div = linha[colunaB] / linha[pivo]; //guarda a divisao da coluna b pela coluna pivo
            if (menorDiv > div) {
              menorDiv = div;
              linhaExit = i; //guarda indice da linha que sai
            } //guarda a menor divisao
          }
        });

        //dividir linha pelo numero em linhaExit*pivo -> precisa ser = 1
        let divisor = this.table[linhaExit][pivo];
        this.table[linhaExit].forEach((col, j) => {
          this.table[linhaExit][j] /= divisor;
        });

        //zerar atoda a coluna pivo exceto a linhaExit
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
      let linha = this.table.length - 1; //pega a ultima linha da tabela
      let x = this.table[linha].find((col) => col < 0); //procura por um valor negativo na linha

      if (x == undefined) return -1; //se nao encontrar nenhum valor retorna -1 -> indica que nao há numeros negativos na ultima linha (solucao otima)

      return this.table[linha].indexOf(x); //se foi encontrado um valor negativo retor a coluna dele sendo necessario ainda fazer operacoes na tabela
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
              this.resposta.x.push(`X${j + 1} = ${value}`);
            }
          } else if (i + 1 == this.table.length - 1) {
            this.resposta.z = `Z = ${this.table[i + 1][j]}`;
          }
        }
      }
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