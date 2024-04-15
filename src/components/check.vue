<script>

//import { onMounted } from 'vue';
import Modell from './../../Modell.json'
import Pruefplaeneverzeichnis from './../../Pruefplaeneverzeichnis.json'
import 'bootstrap/dist/css/bootstrap.css'

import { BarcodeScanner } from '@capacitor-community/barcode-scanner'

import JSZip from 'jszip'

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
        //if (this.scan_checkPermission()) BarcodeScanner.prepare(); //hier ist noch ein Problem im Browser
        //else this.scanNichtVerfuegbar = true;
    },
    methods: {
        schrittAlsHtmlEintragBauen(displayName, schritt, id)
        {
            console.log("schrittAlsHtmlEintragBauen");

            let element = document.getElementById(id + "-schritt-div");
            
            if (!element) {
                element = document.createElement("div");
                element.id = id + "-schritt-div";
                element.classList.add("row");
                element.classList.add("row-col-3");

                element.classList.add('justify-content-md-center');
            }

            element.innerHTML = "";

            let name = document.createElement("div");
            name.classList.add("col-md-2");
            name.classList.add("schritt-titel");
            name.innerText = displayName;

            let content = document.createElement("div");
            content.classList.add("col");

            let add = document.createElement("div");
            add.classList.add("col");
            add.innerText = "+";

            let edit = document.createElement("div");
            edit.classList.add("col");
            edit.innerText = "-";

            let view = document.createElement("div");
            view.classList.add("col");
            view.innerText = "X";

            switch (schritt.Typ)
            {
                case "Foto":
                {
                    content.innerHTML = '<input type="image" id="' + id + '-schritt-div-content">';
                    break;
                }
                case "Text":
                {
                    add.addEventListener("click", function() { vm.startScan(replacementString)});
                    content.innerHTML = '<input type="text" id="' + id + '-schritt-div-content">';
                    break;
                }
                case "Checkbox":
                {
                    content.innerHTML = '<input type="checkbox" id="' + id + '-schritt-div-content"/>';
                    break;
                }
                case "Barcode":
                {
                    break;
                }
                case "Anleitung":
                {
                    content.innerHTML = '<span id="' + id + '-schritt-div-content">' + schritt.Beschreibung + "</span>";
                    break;
                }
            }

            element.appendChild(name);
            element.appendChild(content);
            element.appendChild(add);
            element.appendChild(edit);
            element.appendChild(view);

            return element;
        },
        kategorieAlsHtmlEintragBauen(kategorieName)
        {
            console.log("kategorieAlsHtmlEintragBauen");

            let element = document.getElementById(kategorieName + "-kategorie-div");
        
            if (!element) {
                element = document.createElement("div");
                element.id = kategorieName + "-kategorie-div";
                //element.classList.add("kategorie");
                element.classList.add("container");                
            }            

            for (const [schrittName, schrittInhalt] of Object.entries(this.pruefung.pruefung[kategorieName]))
            {
                if (schrittName == "anzahlSchritte" || schrittName == "erfuellteSchritte") continue;
                let schritt = this.schrittAlsHtmlEintragBauen(schrittName, schrittInhalt, kategorieName + "." + schrittName);
                if (!document.getElementById(kategorieName + "." + schrittName + "-schritt-div")) element.appendChild(schritt);
            }

            return element;
        },
        kategorieEinhaengen(kategorieName)
        {
            let element = document.getElementById("injectionPointKategorie");

            element.innerHTML = "";

            for (const [schrittName, schrittInhalt] of Object.entries(this.pruefung.pruefung[kategorieName]))
            {
                if (schrittName == "anzahlSchritte" || schrittName == "erfuellteSchritte") continue;
                let schritt = this.schrittAlsHtmlEintragBauen(schrittName, schrittInhalt, kategorieName + "." + schrittName);
                if (!document.getElementById(kategorieName + "." + schrittName + "-schritt-div")) element.appendChild(schritt);
            }



        },
        kategorienEinhaengen()
        {
            console.log("kategorienEinhaengen");

            let element = document.getElementById("injectionPointKategorien");

            for (const [kategorieName, kategorieInhalt] of Object.entries(this.pruefung.pruefung))
            {
                if (kategorieName == "Eingangsinformationen") continue;
                let kategorie = this.kategorieAlsHtmlEintragBauen(kategorieName);
                if (!document.getElementById(kategorieName + "-kategorie-div")) element.appendChild(kategorie);
            }
        },
        eingangsinformationenEinhaengen()
        {
            console.log("eingangsinformationenEinhaengen");

            let element = document.getElementById("injectionPointAuftragsinfos");

            let kategorie = this.kategorieAlsHtmlEintragBauen("Eingangsinformationen");
            if (!document.getElementById("Eingangsinformationen-kategorie-div")) element.appendChild(kategorie);
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
                case 3: {
                    this.kategorieErfuellteSchritteUeberpruefen("Eingangsinformationen");
                    return this.checkKategorie("Eingangsinformationen");
                }
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
                //if (!this.vorbereitungenFuerSchritt()) return;
                this.$refs.step3.classList.remove('active');
                this.$refs.step4.classList.add('active');
            }

            if (this.step == 4)
            {
                this.$refs.step4.classList.remove('active');
                this.$refs.step5.classList.add('active');
            }

            if (this.step < 5) ++this.step;

            if (this.step == 3)
            {
                this.eingangsinformationenEinhaengen();

                let element = document.getElementById("Eingangsinformationen-kategorie-div");
                element.style.display = "block";
            }

            if (this.step == 5)
            {
                this.kategorienEinhaengen();
                
                for (const [kategorieName, kategorieInhalt] of Object.entries(this.pruefung.pruefung))
                {
                    let element = document.getElementById(kategorieName + "-kategorie-div");
                    let aktiveKategorie = this.darstellung.aktiveKategorie;
                    if (kategorieName == this.darstellung.aktiveKategorie)
                    {
                        element.style.display = "block";
                    } else {
                        element.style.display = "none";
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
                let element = document.getElementById("Eingangsinformationen-kategorie-div");
                element.style.display = "none";

                this.$refs.step3.classList.remove('active');                
                this.$refs.step2a.classList.add('active');
            }

            if (this.step == 4)
            {
                this.$refs.step4.classList.remove('active');
                this.$refs.step3.classList.add('active');

                let element = document.getElementById("Eingangsinformationen-kategorie-div");
                element.style.display = "block";
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
                    //schrittInhalt["elementId"] = "";
                }
                kategorieInhalt["anzahlSchritte"] = Object.keys(kategorieInhalt).length;
                kategorieInhalt["erfuellteSchritte"] = 0;
            }

            this.pruefung.pruefung = vorbereitet;
        },
        kategorieStarten(kategorieName)
        {
            console.log("kategorieStarten");

            this.darstellung.aktiveKategorie = kategorieName;
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
                if (schrittName == "anzahlSchritte" || schrittName == "erfuellteSchritte") continue;
                schrittInhalt["erfuellt"] = this.schrittUeberpruefen(kategorieName + "." + schrittName, schrittInhalt);
                if (schrittInhalt.erfuellt) ++erfuelltZaehler;
            }

            this.pruefung.pruefung[kategorieName].erfuellteSchritte = erfuelltZaehler;
        },
        schrittUeberpruefen(schrittId, schritt)
        {
            console.log("schrittUeberpruefen");

            let id = schrittId + "-schritt-div-content";
            let element = document.getElementById(id);

            if (!element) return true;

            switch (schritt.Typ)
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

            return true;
        },
        async scan_checkPermission() {
            // check or request permission
            try {
                const status = await BarcodeScanner.checkPermission({ force: true });

                if (status.granted) {
                    // the user granted permission
                    return true;
                }
            } catch {

            }

            return false;
        },
        async scan_startScan() {
            if (this.scanNichtVerfuegbar) return;

            console.log("scan_startScan");
            // make background of WebView transparent
            // note: if you are using ionic this might not be enough, check below
            BarcodeScanner.hideBackground();

            const result = await BarcodeScanner.startScan(); // start scanning and wait for a result

            // if the result has content
            if (result.hasContent) {
                console.log(result.content); // log the raw scanned content
            }
        },
        scan_stopScan() {
            BarcodeScanner.showBackground();
            BarcodeScanner.stopScan();
        },
        uploadFile(url, file) {
            //const url = 'http://www.example.com/upload'; // Replace with your API endpoint

            const formData = new FormData();
            formData.append('file', file);

            fetch(url, {
                method: 'POST',
                body: formData,
                headers: {
                    "Content-Type": "multipart/form-data"
                }
            })
            .then(response => response.json())
            .then(data => {
                console.log('File uploaded successfully:', data);
                // Handle the response from the server
            })
            .catch(error => {
                console.error('Error uploading file:', error);
                // Handle any errors that occur during the upload
            });
        },
        pruefungZwischenspeichern() {
            let zip = new JSZip();

            let elements = document.querySelectorAll("[type='image']");

            if (elements.length == 0) return;

            // Loop through the selected elements
            for (var i = 0; i < elements.length; i++) {
                let element = elements[i];

                if (element.src == '') continue;

                //let kategorie_id = element.id.split('.')[0];
                let name = element.id.split('-')[0];
                
                //let ordner = zip.folder(kategorie_id);
                ordner.file(name, element.src, {base64: true});
            }

            zip.generateAsync({ type: "blob" })
                .then(function (content) {
                    uploadFile(content, "LINK");
                });
        }
    },
    data() {
        return {
            step: 1,
            nextStepMoeglich: true,
            scanNichtVerfuegbar: false,
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
            darstellung: {
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
        <span>Schritt {{step}}</span>
        

        <button class="btn btn-secondary menu-button" type="button" id="dropdownMenuButton1" data-bs-toggle="dropdown" aria-expanded="false">
            Menü
        </button>
        <!-- button @click="scan_startScan">Senden</button -->

    </div>

    <div id="step1_start" class="step active" ref="step1">
        <div class="container text-center">
            <div class="row justify-content-md-center">
                <div class="col col-md-6">
                    <img class="img-fluid" src="/src/assets/FinalesLogo_2022-10_rgb_aufgeräumt_gross.png"/>
                </div>
                <div class="col col-md-6 d-flex align-items-center ">
                    <div class="col-10 d-grid gap-5 mx-auto h-75">
                        <button class="btn btn-lg btn-start fs-1" @click="FunktionNeuePruefung">Neue Prüfung</button>
                        <button class="btn btn-lg btn-start fs-1" @click="FunktionPruefungFortsetzen" disabled style="background-color:gray;opacity:0.5;">Prüfung fortsetzen</button>
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
        

        <div class="text-center h1">
            <div class="mb-5">Prüfauftrag</div>

            <div id="injectionPointAuftragsinfos"/>
        </div>
    </div>

    <div id="step4_uebersicht" class="step" ref="step4">
        <div class="container text-center h1">
            <div class="mb-5">Übersicht</div>
                
            <div class="row">
                <div class="col-8">
                    <div class="row" v-for="(value, key) in pruefung.pruefung" :key="key" :value="value">
                        <button class="btn btn-lg fs-1" style="margin: 5px" v-on:click="kategorieStarten(key)">{{ key }} : {{ value.erfuellteSchritte }}/{{ value.anzahlSchritte }}</button>
                    </div>
                </div>
                <div class="col-4">
                    <div class="row" v-for="(value, key) in darstellung.eingangsinformationen" :key="key" :value="value">
                        <div class="col" :id="'ueberblick.eingangsinformationen.' + key">{{ key }}</div>
                    </div>
                </div>                    
            </div>
        </div>
    </div>

    <div id="step5_pruefung" class="step" ref="step5">
        <div class="text-center h1" id="injectionPointKategorien"></div>
        <div class="text-center h1" id="injectionPointKategorie"></div>
    </div>

    <div class="button-bar mt-5">
      <button v-if="step==1" disabled class="btn btn-lg step-button-left-inactive fs-1">Zurück</button>
      <button @click="prevStep" v-if="step!=1" class="btn btn-lg step-button-left fs-1">Zurück</button>
      
      <button v-if="!nextStepMoeglich" disabled class="btn btn-lg step-button-right-inactive fs-1">Weiter</button>
      <button @click="nextStep" v-if="step<4 && step!=1 && nextStepMoeglich" class="btn btn-lg step-button-right fs-1">Weiter</button>
      <button @click="submit" v-if="step==4" class="btn btn-lg step-button-right fs-1">Senden</button>
      
    </div>



    <button @click="pruefungZwischenspeichern" class="btn btn-lg">Test</button>
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

.step-button-right {
  /*margin-left: auto;*/
  color: white;
  border-color: white;
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




.zeile {
    background-color: #9ba0a3;
    border-radius: 5px;
    height: 3vh;
    width: 80vw;

    
}

.schritt-titel {
    background-color: #9ba0a3;
    border-radius: 5px;
    height: 3vh;
}


</style>
