<template>
  <div id="app">
    <div class="navbar">
      <a class="active" href="kerites.txt">kerites.txt</a>
      <a href="Kerítés_fel.pdf">Feladat.pdf</a>
      <a href="Kerítés_jav.pdf">Javítási.pdf</a>
      <a href="https://github.com/detrikmilan/Kerites">Github</a>
      <a href="https://github.com/detrikmilan">SDK</a>
    </div>
    <TxtReader label="Forrás (kerites.txt):" placeholder="Bedobó mező" @load="txtSorai = $event" />
    <div v-if="mutat" id="megoldas">
      <p>2. feladat<br />Az eladott telkek száma: {{ telkek.length }}</p>
      <p>
        3. feladat<br />A {{ utolsoTelek.oldal }} oldalon adták el az utolsó telket<br />
        Az utolsó telek házszáma: {{ utolsoTelek.hazSzama }}
      </p>
      <p>4. feladat<br />A szomszédossal egyezik a kerítés színe: {{ ugyanOlyanSzinuKerites }}</p>
      <p>
        5. feladat<br />Adjon meg egy házszámot!
        <input v-model="hazszamInputStr" type="number" min="1" max="117" /><br />
        A kerírés színe / állapota: {{ keritesSzineAllapota }}<br />
        Egy lehetséges festési szín: {{ lehetsegesFestesiSzin }}
      </p>
    </div>
    <p v-if="mutat">
      <TxtWriter title="utcakep.txt állomány mentése" :content="utcakep" filename="utcakep.txt" />
    </p>
    <div v-if="mutat" id="egyebek">
      <pre>
utcakep.txt fájl:
{{ utcakep }}
kerites.txt fájl:
{{ txtSorai }}
      </pre>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";
import Telek from "./telek";
import TxtReader from "./components/TxtReader.vue";
import TxtWriter from "./components/TxtWriter.vue";

@Component({ components: { TxtReader, TxtWriter } })
export default class App extends Vue {
  private telkek: Telek[] = [];
  private txtSorai: string = "";
  private mutat: boolean = false;

  private hazszamInputStr: string = "83";

  private utcakep: string = "";

  @Watch("txtSorai", { immediate: true, deep: true })
  private haForrasValtozik(val: string) {
    if (val !== "") this.feldolgoz();
  }

  private feldolgoz(): void {
    try {
      Telek.hazszamReset();
      this.txtSorai.split("\n").forEach(i => {
        const aktSor: string = i.trim();
        if (aktSor.length > 0) this.telkek.push(new Telek(aktSor));
      });
      let sor1: string = "";
      let sor2: string = "";
      for (const i of this.telkek.filter(x => x.paratlanOldali)) {
        sor1 += "".padEnd(i.telekSzelessege, i.keritesSzine);
        sor2 += i.hazSzama.toString().padEnd(i.telekSzelessege, " ");
      }
      this.utcakep = `${sor1}\n${sor2}\n`;

      this.mutat = true;
    } catch (error) {
      this.mutat = false;
      this.telkek = [];
      this.txtSorai = "";
      window.alert("Hibás forrás!");
      location.reload();
    }
  }

  private get utolsoTelek(): Telek {
    return this.telkek[this.telkek.length - 1];
  }

  private get ugyanOlyanSzinuKerites(): number {
    let elozoTelek: Telek;
    for (const aktTelek of this.telkek.filter(x => x.paratlanOldali)) {
      if (
        elozoTelek! &&
        aktTelek.keritesSzine !== "#" &&
        aktTelek.keritesSzine !== ":" &&
        aktTelek.keritesSzine === elozoTelek!.keritesSzine
      ) {
        return elozoTelek!.hazSzama;
      } else elozoTelek = aktTelek;
    }
    return -1;
  }

  private get keritesSzineAllapota(): string {
    const hazszamInput: number = parseInt(this.hazszamInputStr, 10);
    const keresettTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput);
    if (keresettTelek.length !== 0) {
      return keresettTelek[0].keritesSzine;
    } else {
      return "Nincs ilyen házszám!";
    }
  }

  private get lehetsegesFestesiSzin(): string {
    const hazszamInput: number = parseInt(this.hazszamInputStr, 10);
    const keresettTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput);
    const balSzomszedTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput + 2);
    const jobbSzomszedTelek: Telek[] = this.telkek.filter(x => x.hazSzama === hazszamInput - 2);
    let lehetsegesSzinek: string[] = ["A", "B", "C", "D"];
    if (keresettTelek.length !== 0) {
      lehetsegesSzinek = lehetsegesSzinek.filter(x => x !== keresettTelek[0].keritesSzine);
    } else return "Nincs ilyen házszám!";

    if (balSzomszedTelek.length !== 0) {
      lehetsegesSzinek = lehetsegesSzinek.filter(x => x !== balSzomszedTelek[0].keritesSzine);
    }

    if (jobbSzomszedTelek.length !== 0) {
      lehetsegesSzinek = lehetsegesSzinek.filter(x => x !== jobbSzomszedTelek[0].keritesSzine);
    }
    return lehetsegesSzinek[0];
  }
}
</script>

<style>
#app {
  font-family: Courier;
}

#megoldas {
  background-color: lightgrey;
  padding: 0px 10px;
  border-radius: 10px;
  max-width: 600px;
}

a {
  text-decoration: none;
  padding-left: 10px;
}

input[type="number"] {
  background-color: lightgray;
}

pre {
  font-size: 1.1em;
  margin: 0;
}

.navbar {
  width: 100%;
  background-color: #555;
  overflow: auto;
}
.navbar a {
  float: left;
  padding: 12px;
  color: white;
  text-decoration: none;
  font-size: 17px;
  width: 25%;
  text-align: center;
}
.navbar a:hover {
  background-color: #000;
}
.navbar a.active {
  background-color: #4caf50;
}

@media screen and (max-width: 500px) {
  .navbar a {
    float: none;
    display: block;
    width: 100%;
    text-align: left;
  }
}
</style>
