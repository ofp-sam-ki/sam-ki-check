<script>

//import { onMounted } from 'vue';
import Modell from './../../Modell.json'
import Pruefplaeneverzeichnis from './../../Pruefplaeneverzeichnis.json'
import 'bootstrap/dist/css/bootstrap.css' 

import { BarcodeScanner } from '@capacitor-community/barcode-scanner'; 

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
        // if (this.scan_checkPermission()) BarcodeScanner.prepare(); //hier ist noch ein Problem im Browser
        // else this.scanNichtVerfuegbar = true; 
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
                
                /* hier Styling */
                element.classList.add('justify-content-md-center');
            }
                
                //element.style.height = "60%";
                //element.style.overflowY = "scroll";

            element.innerHTML = "";

            let name = document.createElement("div");
            name.classList.add("col-md-2");
            name.classList.add("schritt-titel");
            name.innerText = displayName;

            let content = document.createElement("div");
            content.classList.add("col");

            let add = document.createElement("div");
            add.classList.add("col");

            let edit = document.createElement("div");
            edit.classList.add("col");
            /* edit.innerText = "-"; */

            let view = document.createElement("div");
            view.classList.add("col");
            view.innerHTML = '<button class="btn btn-outline-secondary btn-lg menu-button me-md-2" type="button">  <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" class="bi bi-x-lg" viewBox="0 0 16 16"> <path d="M2.146 2.854a.5.5 0 1 1 .708-.708L8 7.293l5.146-5.147a.5.5 0 0 1 .708.708L8.707 8l5.147 5.146a.5.5 0 0 1-.708.708L8 8.707l-5.146 5.147a.5.5 0 0 1-.708-.708L7.293 8z"/> </svg></button> ';
            /*view.onclick = delete content; */

            switch (schritt.Typ)
            {
                case "Foto":
                {
                    content.innerHTML = '<input type="image" id="' + id + '-schritt-div-content">';
                    add.innerHTML = '<button class="btn btn-outline-secondary btn-lg menu-button me-md-2" type="button"> <svg xmlns="http://www.w3.org/2000/svg" width="25" height="25" fill="currentColor" class="bi bi-camera" viewBox="0 0 16 16"> <path d="M15 12a1 1 0 0 1-1 1H2a1 1 0 0 1-1-1V6a1 1 0 0 1 1-1h1.172a3 3 0 0 0 2.12-.879l.83-.828A1 1 0 0 1 6.827 3h2.344a1 1 0 0 1 .707.293l.828.828A3 3 0 0 0 12.828 5H14a1 1 0 0 1 1 1zM2 4a2 2 0 0 0-2 2v6a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V6a2 2 0 0 0-2-2h-1.172a2 2 0 0 1-1.414-.586l-.828-.828A2 2 0 0 0 9.172 2H6.828a2 2 0 0 0-1.414.586l-.828.828A2 2 0 0 1 3.172 4z"/> <path d="M8 11a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5m0 1a3.5 3.5 0 1 0 0-7 3.5 3.5 0 0 0 0 7M3 6.5a.5.5 0 1 1-1 0 .5.5 0 0 1 1 0"/> </svg> </button>' ;
                    
                    add.addEventListener("click", function() {
                            // Überprüfe, ob die getUserMedia-API verfügbar ist
                            if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
                            // Öffne die Kamera
                            navigator.mediaDevices.getUserMedia({ video: true })
                            .then(function(stream) {
                            // Erfolgreich die Kamera geöffnet
                            // Nutze den Stream, um das Video-Element anzuzeigen oder für andere Zwecke
                            })
                            .catch(function(error) {
                                // Fehler beim Öffnen der Kamera
                                console.error('Fehler beim Öffnen der Kamera:', error);
                            });
                        } else {
                            // Die getUserMedia-API wird nicht unterstützt
                            console.error('getUserMedia wird nicht unterstützt');
                        }
                        });
                    
                    /*add.addEventListener("change", element.classList.add('table-success')); */
                    
                    let foto = document.createElement("input");
                    foto.type = "image";
                    foto.id = "' + id + '-schritt-div-content";

                    foto.addEventListener("input", function()
                        {if (foto = empty) 
                            {element.classList.add('table-success');} 
                        else {element.classList.remove('table-success');}
                    });    
                    break;
                }
                case "Text":
                {
                    add.addEventListener("click", function() { vm.startScan(replacementString)});
                    
                    add.innerHTML = '<button class="btn btn-outline-secondary btn-lg menu-button me-md-2" type="button"><svg xmlns="http://www.w3.org/2000/svg" width="25" height="25" fill="currentColor" class="bi bi-pen" viewBox="0 0 16 16"> <path d="m13.498.795.149-.149a1.207 1.207 0 1 1 1.707 1.708l-.149.148a1.5 1.5 0 0 1-.059 2.059L4.854 14.854a.5.5 0 0 1-.233.131l-4 1a.5.5 0 0 1-.606-.606l1-4a.5.5 0 0 1 .131-.232l9.642-9.642a.5.5 0 0 0-.642.056L6.854 4.854a.5.5 0 1 1-.708-.708L9.44.854A1.5 1.5 0 0 1 11.5.796a1.5 1.5 0 0 1 1.998-.001m-.644.766a.5.5 0 0 0-.707 0L1.95 11.756l-.764 3.057 3.057-.764L14.44 3.854a.5.5 0 0 0 0-.708z"/> </svg></button> ' ;
                    
                    let text = document.createElement("input");
                    text.type = "text";
                    text.id = "' + id + '-schritt-div-content";

                    text.addEventListener("input", function()
                        {if (text.value.trim() === "") 
                            {element.classList.remove('table-success');} 
                        else {element.classList.add('table-success');}
                    });
                    
                    view.addEventListener("click", function()
                                {text.value = "";
                                 element.classList.remove('table-success');});
                    /*content.innerHTML = '<input type="text" id="' + id + '-schritt-div-content">'; */
                    
                    content.appendChild(text);
                    break;
                }
                case "Checkbox":
                {
                    /* content.innerHTML = '<input type="checkbox" st id="' + id + '-schritt-div-content" name="checkbox" />'; */

                    let checkbox = document.createElement("input");
                    checkbox.type = "checkbox";
                    checkbox.id = "' + id + '-schritt-div-content";
                    checkbox.style.transform = "scale(2)";


                    checkbox.addEventListener("change", function()
                        {if (checkbox.checked) 
                            {element.classList.add('table-success');} 
                        else {element.classList.remove('table-success');}
                    });

                    view.addEventListener("click", function()
                                {checkbox.checked = false;
                                 element.classList.remove('table-success');});

                    content.appendChild(checkbox);
                    break;
                }
                case "Barcode":
                {
                    add.innerHTML = '<button class="btn btn-outline-secondary btn-lg menu-button me-md-2" type="button"> <svg xmlns="http://www.w3.org/2000/svg" width="25" height="25" fill="currentColor" class="bi bi-upc-scan" viewBox="0 0 16 16"> <path d="M1.5 1a.5.5 0 0 0-.5.5v3a.5.5 0 0 1-1 0v-3A1.5 1.5 0 0 1 1.5 0h3a.5.5 0 0 1 0 1zM11 .5a.5.5 0 0 1 .5-.5h3A1.5 1.5 0 0 1 16 1.5v3a.5.5 0 0 1-1 0v-3a.5.5 0 0 0-.5-.5h-3a.5.5 0 0 1-.5-.5M.5 11a.5.5 0 0 1 .5.5v3a.5.5 0 0 0 .5.5h3a.5.5 0 0 1 0 1h-3A1.5 1.5 0 0 1 0 14.5v-3a.5.5 0 0 1 .5-.5m15 0a.5.5 0 0 1 .5.5v3a1.5 1.5 0 0 1-1.5 1.5h-3a.5.5 0 0 1 0-1h3a.5.5 0 0 0 .5-.5v-3a.5.5 0 0 1 .5-.5M3 4.5a.5.5 0 0 1 1 0v7a.5.5 0 0 1-1 0zm2 0a.5.5 0 0 1 1 0v7a.5.5 0 0 1-1 0zm2 0a.5.5 0 0 1 1 0v7a.5.5 0 0 1-1 0zm2 0a.5.5 0 0 1 .5-.5h1a.5.5 0 0 1 .5.5v7a.5.5 0 0 1-.5.5h-1a.5.5 0 0 1-.5-.5zm3 0a.5.5 0 0 1 1 0v7a.5.5 0 0 1-1 0z"/> </svg></button> ' ;
                    
                    add.addEventListener("click", function() {
                            // Überprüfe, ob die getUserMedia-API verfügbar ist
                            if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
                            // Öffne die Kamera
                            navigator.mediaDevices.getUserMedia({ video: true })
                            .then(function(stream) {
                            // Erfolgreich die Kamera geöffnet
                            // Nutze den Stream, um das Video-Element anzuzeigen oder für andere Zwecke
                            })
                            .catch(function(error) {
                                // Fehler beim Öffnen der Kamera
                                console.error('Fehler beim Öffnen der Kamera:', error);
                            });
                        } else {
                            // Die getUserMedia-API wird nicht unterstützt
                            console.error('getUserMedia wird nicht unterstützt');
                        }
                        });
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

            let table = document.createElement("table");

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


                let row = document.createElement("tr");
                let title = document.createElement("td");
                title.innerHTML = "<p>" + key + "</p>";
                row.appendChild(title);

                let desc = document.createElement("td");
                desc.innerHTML = "<p>" + value.Beschreibung + "</p>";
                row.appendChild(desc);
                
                let content = document.createElement("td"); /*Bedingung, nur wenne s keine Anleitung ist*/

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
        /*Funktion für Weiter-Button */
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
                /* if (!this.vorbereitungenFuerSchritt()) return; */
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
            }

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
    <!-- Header -->
    <div class="menubar mt-3 mb-5">
        <a class="navbar-brand" href="step1_start">
            <img src="/src/assets/FinalesLogo_2022-10_rgb_aufgeräumt_gross.png" alt="Logo" width="120" class="d-inline-block align-text-top">
        </a>
        <h1 class="me-md-4">SAM-KI-Check</h1>
        <h2 class="me-md-4">Schritt {{step}}</h2>

        <button class="btn btn-secondary btn-lg menu-button me-md-2" type="button" id="dropdownMenuButton1" data-bs-toggle="dropdown" aria-expanded="false">
            <svg xmlns="http://www.w3.org/2000/svg" width="30" height="30" fill="white" class="bi bi-list" viewBox="0 0 16 16">
                    <path fill-rule="evenodd" d="M2.5 12a.5.5 0 0 1 .5-.5h10a.5.5 0 0 1 0 1H3a.5.5 0 0 1-.5-.5m0-4a.5.5 0 0 1 .5-.5h10a.5.5 0 0 1 0 1H3a.5.5 0 0 1-.5-.5m0-4a.5.5 0 0 1 .5-.5h10a.5.5 0 0 1 0 1H3a.5.5 0 0 1-.5-.5"/>
            </svg>
        </button>

        <button class="btn btn-outline-secondary btn-lg btn-light" type="button" @click="scan_startScan">Senden</button>
        
    </div>

    <!--  Home-Screen: Auswahl Prüfpläne (neu oder zwischengespeichert) -->
    <div id="step1_start" class="step active" ref="step1">
        <div class="container text-center">
            <div class="row justify-content-md-center">
                <div class="col col-md-5">
                    <img class="img-fluid" src="/src/assets/FinalesLogo_2022-10_rgb_aufgeräumt_gross.png"/>
                </div>
                <div class="col col-md-6 d-flex align-items-center">
                    <div class="col-10 d-grid gap-4 mx-auto h-50">
                        <button type="button" class="btn btn-lg fs-1 btn-secondary" @click="FunktionNeuePruefung">Neue Prüfung</button>
                        <button type="button" class="btn btn-lg fs-1 btn-outline-secondary" @click="FunktionPruefungFortsetzen">Prüfung fortsetzen</button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Neuen Prüfplan starten -->
    <div id="step2a_pruefplan" class="step" ref="step2a">
        <div class="container text-center h1">
            <h1>Prüfplan auswählen</h1>
            
            <div>
                <select class="form-select form-select-lg mb-3" size="15" v-model="check.pruefplanId" id="selectionPruefplan" ref="auswahlPruefplan">
                    <option v-for="(value, key) in check.pruefplane_lesbar" :key="key" :value="value">
                        {{ key }} für {{ value }}
                    </option>
                </select>
            </div>
        </div>
    </div>


    <!-- Gespeicherte Prüfung fortsetzen -->
    <div id="step2b_fortsetzen" class="step" ref="step2b">
        <div class="container text-center h1">
            <h1>Prüfung fortsetzen</h1>
            
            <div>
                <select class="form-select" size="10" v-model="check.gespeicherte_pruefungId" id="selectionGespeichertePruefung">
                    <option v-for="(value, key) in check.gespeicherte_pruefungen" :key="key" :value="value">
                        {{ key }}
                    </option>
                </select>
            </div>
        </div>
    </div>

    <!-- Prüfauftrag -->
    <div id="step3_auftragsinfos" class="step" ref="step3">
        <div >
            <div class="mb-5 h1 text-center">Prüfauftrag</div>

            <table class="table align-middle" id="injectionPointAuftragsinfos">
            </table>
        </div>
    </div>

    <!-- Übersicht des ausgewählten Prüfplans -->
    <div id="step4_uebersicht" class="step" ref="step4">
        <div class="container text-center h1">
            <div class="mb-5">Übersicht</div>
                
            <div class="container">
                <div class="row">
                    <div class="col">
                        <button v-for="(value, key) in pruefung.pruefung" :key="key" :value="value" class="btn btn-lg" style="margin: 5px" v-on:click="kategorieStarten(key)">{{ key }}: {{ value.erfuellteSchritte }}/{{ value.anzahlSchritte }}</button>
                    </div>
                    <div class="col">
                    <!-- erstmal Dummy-Daten -->
                        <div class="row" v-for="(value, key) in darstellung.eingangsinformationen" :key="key" :value="value">
                            <div class="col" :id="'ueberblick.eingangsinformationen.' + key">{{ key }}</div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div id="step5_pruefung" class="step" ref="step5">
        <div class="mb-5 h1 text-center">Schritt {{step}}</div>
        <div class="table" id="injectionPointKategorien">
        </div>
    </div>

    <!-- Weiter und Zurück Buttons -->
    <div class="m-3">
      <button type="button" v-if="step==1" disabled class="btn btn-outline-secondary btn-lg step-button-left-inactive fs-1 position-absolute bottom-0 start-0">Zurück</button>
      <button type="button" @click="prevStep" v-if="step!=1" class="btn btn-outline-secondary btn-lg step-button-left fs-1 position-absolute bottom-0 start-0">Zurück</button>
      
      <button type="button" v-if="!nextStepMoeglich" disabled class="btn btn-secondary btn-lg step-button-right-inactive fs-1 position-absolute bottom-0 end-0 translate-middle-x">Weiter</button>
      <button type="button" @click="nextStep" v-if="step<4 && step!=1 && nextStepMoeglich" class="btn btn-secondary btn-lg step-button-right fs-1 position-absolute bottom-0 end-0 translate-middle-x">Weiter</button>
      <button type="button" @click="submit" v-if="step==4" class="btn btn-primary btn-lg step-button-right fs-1 position-absolute bottom-0 end-0 translate-middle-x">Senden</button>
      
    </div>



   <!--  <div id="checkInjectionPoint">hello</div> -->
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


.button-bar {
  display:flex;
  justify-content: space-between;
  /*position: absolute;*/
  bottom: 10px;
  /*position absolute */
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
  /* background-color: #005b7f; */
  /*margin-right: 7vw;*/
  margin-left: auto;
  margin-bottom: 1.5vw;
  /*bottom:0;*/
  /* box-shadow: 3px 3px 5px black; */
}

.step-button-right {
  /*margin-left: auto;*/
  /* color: white; */
  /* border-color: white; */
  /*margin-right: 7vw;*/
  margin-left: auto;
  margin-bottom: 1.5vw;
  bottom:0;
  /* box-shadow: 3px 3px 5px black; */
}

.step-button-left {
  margin-left: 7vw;
  margin-right: auto;
  margin-bottom: 1.5vw;
  bottom:0;
  /* box-shadow: 3px 3px 5px black; */
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
