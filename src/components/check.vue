<script>

//import { onMounted } from 'vue';
import Modell from './../../Modell.json'
import Pruefplaeneverzeichnis from './../../Pruefplaeneverzeichnis.json'
import 'bootstrap/dist/css/bootstrap.css'

/*onMounted(() => {
    createCheckList();
})*/

export default {
    beforeMount() {
        //this.createCheckList();
        for (const [key, value] of Object.entries(this.check.pruefplaene))
        {
            this.check.pruefplane_lesbar[key] = "";
            for (const pruefplanname in value)
            {
                this.check.pruefplane_lesbar[key] += value[pruefplanname] + ", "
            }
            this.check.pruefplane_lesbar[key] = this.check.pruefplane_lesbar[key].substring(0, this.check.pruefplane_lesbar[key].length - 2);
        }
        this.modellVorbereiten();
    },
    methods: {
        schrittAlsHtmlEintragBauen(schritt, id)
        {
            console.log("schrittAlsHtmlEintragBauen");

            let element = document.getElementById(id + "-schritt-div");
            
            if (!element) {
                element = document.createElement("div");
                element.id = id + "-schritt-div";
            }

            switch (schritt.Typ)
            {
                case "Foto":
                {
                    element.innerHTML = '<img id="' + id + '" style="max-width: 300px;max-height: 300px">';
                    break;
                }
                case "Text":
                {
                    //element.innerHTML = element + ": " + this.pruefung[id_im_datenmodell];
                    break;
                }
                case "Checkbox":
                {
                    element.innerHTML = '<input type="checkbox" id="' + id + '">';
                    break;
                }
                case "Barcode":
                {
                    break;
                }
                case "Anleitung":
                {
                    element.innerHTML = element + ": " + schritt.Beschreibung;
                    break;
                }
            }

            return element;
        },
        kategorieAlsHtmlEintragBauen(kategorieName)
        {
            console.log("kategorieAlsHtmlEintragBauen");

            let element = document.getElementById(kategorieName + "-kategorie-div");
        
            if (!element) {
                element = document.createElement("div");
                element.id = kategorieName + "-kategorie-div";
            }            

            for (const [schrittName, schrittInhalt] of Object.entries(this.pruefung.pruefung[kategorieName]))
            {
                let schritt = this.schrittAlsHtmlEintragBauen(schrittInhalt, kategorieName + "." + schrittName);
                if (!document.getElementById(schrittName + "-schritt-div")) element.appendChild(schritt);
            }

            return element;
        },
        kategorienEinhaengen()
        {
            console.log("kategorienEinhaengen");

            let element = document.getElementById("injectionPointKategorien");

            for (const [kategorieName, kategorieInhalt] of Object.entries(this.pruefung.pruefung))
            {
                let kategorie = this.kategorieAlsHtmlEintragBauen(kategorieName);
                if (!document.getElementById(kategorieName + "-kategorie-div")) element.appendChild(kategorie);
            }
        },
        FunktionNeuePruefung()
        {
            this.nextStep();
            this.$refs.step1.classList.remove('active');
            this.$refs.step2a.classList.add('active');
        },
        FunktionPruefungFortsetzen()
        {
            this.nextStep();
        },
        modellAus()
        {
            var attachmentPoint = document.getElementById('checkInjectionPoint');

            let table = document.create("table");

            var vm = this;

            for (const [key, value] of Object.entries(Modell))
            {
                /*let attachment = document.createElement('div');

                let title = document.createElement("div");
                title.innerHTML = "<p>" + key + "</p>";
                title.classList.add("div_reihe");
                attachment.appendChild(title);

                let desc = document.createElement("div");
                desc.innerHTML = "<p>" + value["Beschreibung"] + "</p>";
                desc.classList.add("div_reihe");
                attachment.appendChild(title);*/


                let row = document.create("tr");
                let title = document.create("td");
                title.innerHTML = "<p>" + key + "</p>";
                row.appendChild(title);

                let desc = document.create("td");
                desc.innerHTML = "<p>" + value.Beschreibung + "</p>";
                row.appendChild(desc);
                
                let content = document.create("td");

                switch (value["Typ"])
                {
                    case "Anleitung": break;
                    case "Foto":
                    {
                        let container = document.createElement("div");
                        container.innerHTML = '<input type="file" accept="image/*" capture="camera" id="camera-' + key + '">';

                        let vorschau = document.createElement("div");
                        vorschau.innerHTML = '<img id="foto-' + key + '" style="max-width: 300px;max-height: 300px">';
                        vorschau.classList.add("div_reihe");
                        container.appendChild(vorschau);

                        let kamera = document.createElement("div");
                        kamera.innerHTML = "<p>Aufnehmen</p>";
                        kamera.classList.add("div_reihe");
                        kamera.addEventListener("onclick", function(){ vm.takeFoto('camera-' + key) });
                        container.appendChild(kamera);

                        attachment.appendChild(container);


                        break;
                    }
                    case "Checkbox":
                    {
                        break;
                    }
                    case "text":
                    {
                        break;
                    }
                }

                row.appendChild(content);

                attachmentPoint.appendChild(attachment);
            }
        },
        takeFoto(key) {
            const input = document.getElementById(key);
            input.click();
        },
        vorbereitungenFuerSchritt() {
            console.log("vorbereitungenFuerSchritt");

            switch (this.step)
            {
                case 1: break;
                case 2: break;
                case 4: this.kategorienUeberpruefen();
                default: break;
            }

        },
        nextStep() {
            console.log("nextstep");
            console.log(this.step);

            if (this.step == 2)
            {
                if (this.$refs.auswahlPruefplan.value == "") return;
                this.$refs.step2a.classList.remove('active');
                this.$refs.step3.classList.add('active');
            }

            if (this.step == 3)
            {
                this.$refs.step3.classList.remove('active');
                this.$refs.step4.classList.add('active');
            }

            if (this.step == 4)
            {
                this.$refs.step4.classList.remove('active');
                this.$refs.step5.classList.add('active');
            }

            if (this.step < 5) ++this.step;

            if (this.step == 5)
            {
                this.kategorienEinhaengen();
                
                for (const [kategorieName, kategorieInhalt] of Object.entries(this.pruefung.pruefung))
                {
                    let element = document.getElementById(kategorieName + "-kategorie-div");
                    if (kategorieName == this.darstellung.aktiveKategorie)
                    {
                        element.classList.add("active");
                    } else {
                        element.classList.remove("active");
                    }
                }
            }
        },
        prevStep() {
            console.log("prevStep");

            if (this.step == 2)
            {
                this.$refs.step2a.classList.remove('active');
                this.$refs.step1.classList.add('active');
            }

            if (this.step == 3)
            {
                this.$refs.step3.classList.remove('active');
                this.$refs.step2a.classList.add('active');
            }

            if (this.step == 4)
            {
                this.$refs.step4.classList.remove('active');
                this.$refs.step3.classList.add('active');
            }

            if (this.step == 5)
            {
                this.$refs.step5.classList.remove('active');
                this.$refs.step4.classList.add('active');
            }

            if (this.step > 1) --this.step;
        },
        submit() {
            console.log("submit");

            if (!this.checkAlle)
            {
                console.log("submit geht it")
            }
        },
        checkAlle() {
            console.log("checkAlle");

            this.kategorienUeberpruefen();

            let res = true;

            for (const [key, value] of Object.entries(Modell))
            {
                res = res && this.checkKategorie(key);
                if (!res) return false;
            }
            return true;
        },
        checkKategorie(kategorieName) {
            console.log("kategorieName");

            return this.pruefung.pruefung[kategorieName].erfuellteSchritte == this.pruefung.pruefung[kategorieName].anzahlSchritte;
        },
        modellVorbereiten()
        {
            console.log("modellVorbereiten");

            var vorbereitet = this.check.model;

            for (var [kategorieName, kategorieInhalt] of Object.entries(vorbereitet))
            {
                for (var [schrittName, schrittInhalt] of Object.entries(kategorieInhalt))
                {
                    schrittInhalt["erfuellt"] = false;
                    schrittInhalt["elementId"] = "";
                }
                kategorieInhalt["anzahlSchritte"] = Object.keys(kategorieInhalt).length;
                kategorieInhalt["erfuellteSchritte"] = 0;
            }

            this.pruefung.pruefung = vorbereitet;
        },
        kategorieStarten(kategorieName)
        {
            console.log("kategorieStarten");

            this.pruefung.aktiveKategorie = kategorieName;
            this.nextStep();
        },
        kategorienUeberpruefen()
        {
            console.log("kategorienUeberpruefen");

            for (var [kategorieName, kategorieInhalt] of Object.entries(this.pruefung.pruefung))
            {
                this.kategorieErfuellteSchritteUeberpruefen(kategorieName);
            }
        },
        kategorieErfuellteSchritteUeberpruefen(kategorieName)
        {
            console.log("kategorieErfuellteSchritteUeberpruefen");

            var erfuelltZaehler = 0;
            for (var [schrittName, schrittInhalt] of Object.entries(this.pruefung.pruefung[kategorieName]))
            {
                schrittInhalt.erfuellt = schrittUeberpruefen(schrittInhalt);
                if (schrittInhalt.erfuellt) ++erfuelltZaehler;
            }

            this.pruefung.pruefung[kategorieName].erfuellteSchritte = erfuelltZaehler;
        },
        schrittUeberpruefen(schritt)
        {
            console.log("schrittUeberpruefen");

            try {
                let element = document.getElementById(schritt["elementId"]);
            } catch {
                return false;
            }

            switch (value["Typ"])
            {
                case "Anleitung": return true;
                case "Foto":
                {
                    if (element.src)
                        if (element.src != "") return true;
                    return false;
                }
                case "Checkbox":
                {
                    return true;
                }
                case "text":
                {
                    if (element.value != "") return true;
                    return false;
                }
            }
        }
    },
    data() {
        return {
            step: 1,
            check: {
                model: Modell,
                pruefplaene: Pruefplaeneverzeichnis,
                pruefplane_lesbar: {},
                pruefplanId: "",
                gespeicherte_pruefungen: {
                    pruefung1: {},
                    pruefung2: {},
                    pruefung3: {},
                    pruefung4: {},
                    pruefung5: {},
                    pruefung6: {}
                },
                gespeicherte_pruefungId: ""
            },
            pruefung: {
                pruefplanId: "",
                pruefung: null,
            },
            darstellung:
            {
                aktiveKategorie: "",
                kategorien: {
                    mechanisch: [0, 10],
                    elektrisch: [5, 8]
                },
                eingangsinformationen: {
                    Schritt1: {
                        Beschreibung: "abc"
                    },
                    Schritt4: {
                        Beschreibung: "abc"
                    }
                }
            }
        }
    }
}

</script>


<template>
    <div class="menubar mt-3 mb-5">
        <span>SAM-KI-Check</span>
        

        <button class="btn btn-secondary menu-button" type="button" id="dropdownMenuButton1" data-bs-toggle="dropdown" aria-expanded="false">
            Menü
        </button>

    </div>

    <div id="step1_start" class="step active" ref="step1">
        <div class="container text-center">
            <div class="row justify-content-md-center">
                <div class="col col-md-6">
                    <img class="img-fluid" src="src/assets/FinalesLogo_2022-10_rgb_aufgeräumt_gross.png"/>
                </div>
                <div class="col col-md-6 d-flex align-items-center ">
                    <div class="col-10 d-grid gap-5 mx-auto h-75">
                        <button class="btn btn-lg btn-start fs-1" @click="FunktionNeuePruefung">Neue Prüfung</button>
                        <button class="btn btn-lg btn-start fs-1" @click="FunktionPruefungFortsetzen">Prüfung fortsetzen</button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div id="step2a_pruefplan" class="step" ref="step2a">
        <div class="container text-center h1">
            <div class="mb-5">Prüfplan auswählen</div>
            
            <div>
                <select class="form-select" size="10" v-model="check.pruefplanId" id="selectionPruefplan" ref="auswahlPruefplan">
                    <option v-for="(value, key) in check.pruefplane_lesbar" :key="key" :value="value">
                        {{ key }} für {{ value }}
                    </option>
                </select>
            </div>
        </div>
    </div>



    <div id="step2b_fortsetzen" class="step" ref="step2b">
        <div class="container text-center h1">
            <div class="mb-5">Prüfung fortsetzen</div>
            
            <div>
                <select class="form-select" size="10" v-model="check.gespeicherte_pruefungId" id="selectionGespeichertePruefung">
                    <option v-for="(value, key) in check.gespeicherte_pruefungen" :key="key" :value="value">
                        {{ key }}
                    </option>
                </select>
            </div>
        </div>
    </div>

    <div id="step3_auftragsinfos" class="step" ref="step3">
    </div>

    <div id="step4_uebersicht" class="step" ref="step4">
        <div class="container text-center h1">
            <div class="mb-5">Übersicht</div>
                
            <div>
                <div class="row">
                    <div class="col-8">
                        <button v-for="(value, key) in pruefung.pruefung" :key="key" :value="value" class="btn btn-lg fs-1" style="margin: 5px" v-on:click="kategorieStarten(key)">{{ key }} : {{ value.erfuellteSchritte }}/{{ value.anzahlSchritte }}</button>
                    </div>
                    <div class="col-4">
                        <div class="row" v-for="(value, key) in darstellung.eingangsinformationen" :key="key" :value="value">
                            <div class="col" :id="'ueberblick.eingangsinformationen.' + key">{{ key }}</div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div id="step5_pruefung" class="step" ref="step5">
        <div class="container text-center h1" id="injectionPointKategorien">
        </div>
    </div>

    <div class="button-bar mt-5">
      <button v-if="step==1" disabled class="btn btn-lg step-button-left-inactive fs-1">Zurück</button>
      <button @click="prevStep" v-if="step!=1" class="btn btn-lg step-button-left fs-1">Zurück</button>
      

      <button @click="nextStep" v-if="step<4 && step!=1" class="btn btn-lg step-button-right fs-1">Weiter</button>
      <button @click="submit" v-if="step==4" class="btn btn-lg step-button-right fs-1">Senden</button>
      
    </div>



    <div id="checkInjectionPoint">hello</div>
</template>


<style scoped>
.menubar {
  height: 17vh;
  width: 100vw;
  background-color: #bbcbd5;

  margin-left: 0vw;

  display: flex;
  align-items: center; /* Centers items vertically */

  color: black;
  font-size: 5vh;
  padding-left: 3vw;
  padding-right: 3vw;

  font-weight: bold;
}

.menu-button {
  margin-left: auto;
  /*margin-right: 3vw;*/
}

.menubar {
  height: 17vh;
  width: 100vw;
  background-color: #bbcbd5;

  margin-left: 0vw;

  display: flex;
  align-items: center; /* Centers items vertically */

  color: black;
  font-size: 5vh;
  padding-left: 3vw;
  padding-right: 3vw;

  font-weight: bold;
}

.button-bar {
  display:flex;
  justify-content: space-between;
}

.step {
  display: none;
}

.step.active {
  display: inline;
}






.btn-start {
    background-color: #005b7f;
    color: white;
    box-shadow: 2px 2px 3px black;
}




.step-button-right {
  /*margin-left: auto;*/
  background-color: #005b7f;
  margin-right: 7vw;
  margin-left: auto;
  margin-bottom: 1.5vw;
  bottom:0;
  box-shadow: 3px 3px 5px black;
}

.step-button-left {
  margin-left: 7vw;
  margin-right: auto;
  margin-bottom: 1.5vw;
  bottom:0;
  box-shadow: 3px 3px 5px black;
}

.step-button-left-inactive {
  margin-left: 7vw;
  margin-right: auto;
  bottom:0;
  color: white;
  border-color: white;
}





</style>
