<script>

//import { onMounted } from 'vue';
import Modell from './../../Modell.json'
//import Pruefplaeneverzeichnis from './../../Pruefplaeneverzeichnis.json'
import 'bootstrap/dist/css/bootstrap.css' 
import 'bootstrap/dist/js/bootstrap.bundle.min.js' //--> HTML Part 
import axios from 'axios';
import { BarcodeScanner } from '@capacitor-community/barcode-scanner' 
import { reactive } from 'vue';
import JSZip from 'jszip'

export default {
    
    beforeMount() {
        document.addEventListener("DOMContentLoaded", function(){
            var popoverTriggerList = [].slice.call(document.querySelectorAll('[data-bs-toggle="popover"]'));
            var popoverList = popoverTriggerList.map(function(element){
                return new bootstrap.Popover(element);
            });
        });
        console.log('Client für SAM-KI-Check');
        
        //Prüfplanverzeichnis initial laden
        axios.get('http://localhost:4000/Pruefplaeneverzeichnis_Test')
        .then((response) => {
            console.log('http://localhost:4000/Pruefplaeneverzeichnis_Test')
            this.check.pruefplaene = response.data;
            for (const [key, value] of Object.entries(this.check.pruefplaene))
            {
                this.check.pruefplane_lesbar[key] = "";
                for (const pruefplanname in value)
                {
                    this.check.pruefplane_lesbar[key] += value[pruefplanname] + ", "
                }
                this.check.pruefplane_lesbar[key] = this.check.pruefplane_lesbar[key].substring(0, this.check.pruefplane_lesbar[key].length - 2);
            }
        })
        .catch(function (error) {
            console.log('error mess');
            console.error(error);
        });

    },
    methods: {
        devideString(input)
        {
            var dotIndex = input.indexOf('.');
            if (dotIndex !== -1)
            {
                var firstWord = input.substring(0, dotIndex);
                return firstWord.trim();
            }
            return '';
        },

        setSettings()
        {
            console.log("Settings")
            this.$refs.step1.classList.remove('active');
            this.$refs.step1b.classList.add('active');
            console.log("Settings2")
        },
        
        schrittAlsHtmlEintragBauen(displayName, schritt, id)
        {
            var firstID= this.devideString(id);
            

            let element = document.getElementById(id + "-schritt-div");
            var userInput = '';
            
            if (!element) {
                element = document.createElement("div");
                element.id = id + "-schritt-div";
                element.classList.add("row");
                element.classList.add("row-col-3");
                
                /* hier Styling */
                element.classList.add('justify-content-md-center', 'pb-3');
            }

            element.innerHTML = "";
           
            //Prüf-Nr.
            let name = document.createElement("div");
            name.classList.add("col-md-2");
            name.classList.add("schritt-titel");
            name.innerText = displayName;

            //Prüfschritt
            let content = document.createElement("div");
            content.classList.add("col");

            //leer?
            let edit = document.createElement("div");
            edit.classList.add("col");

            //Buttons für Kamera / Barcode
            let add = document.createElement("div");
            add.classList.add("col");

            //Checkbox oder Inhalt löschen
            let view = document.createElement("div");
            view.classList.add("col");

            //Kommentare
            let comm = document.createElement("div");
            comm.classList.add("col");

            switch (schritt.Typ)
            {
                case "Foto":
                    {
                        content.innerHTML = '<input type="image" id="' + id + '-schritt-div-content">';
                        add.innerHTML = '<button class="btn btn-outline-secondary btn-lg menu-button me-md-2" type="button"> <svg xmlns="http://www.w3.org/2000/svg" width="25" height="25" fill="currentColor" class="bi bi-camera" viewBox="0 0 16 16"> <path d="M15 12a1 1 0 0 1-1 1H2a1 1 0 0 1-1-1V6a1 1 0 0 1 1-1h1.172a3 3 0 0 0 2.12-.879l.83-.828A1 1 0 0 1 6.827 3h2.344a1 1 0 0 1 .707.293l.828.828A3 3 0 0 0 12.828 5H14a1 1 0 0 1 1 1zM2 4a2 2 0 0 0-2 2v6a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V6a2 2 0 0 0-2-2h-1.172a2 2 0 0 1-1.414-.586l-.828-.828A2 2 0 0 0 9.172 2H6.828a2 2 0 0 0-1.414.586l-.828.828A2 2 0 0 1 3.172 4z"/> <path d="M8 11a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5m0 1a3.5 3.5 0 1 0 0-7 3.5 3.5 0 0 0 0 7M3 6.5a.5.5 0 1 1-1 0 .5.5 0 0 1 1 0"/> </svg> </button>';

                        var vm = this;
                        
                        add.addEventListener("click", function () {
                            // Überprüfe, ob die getUserMedia-API verfügbar ist
                            if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
                                console.log("mediaDevices nicht verfügbar");
                                return;
                            }
                            navigator.mediaDevices.getUserMedia({ video: true })
                                .then(function (stream) {
                                    console.log("Starte Webcam-Stream")
                                    let video = document.createElement("video");
                                    video.id = "fotoaufnahme";
                                    video.srcObject = stream;
                                    console.log(video.srcObject);
                                    video.autoplay = true;
                                    video.style.position = "fixed";
                                    video.style.top = "50%";
                                    video.style.left = "50%";
                                    video.style.transform = "translate(-50%, -50%)";

                                    video.innerHTML = '';

                                    let canvas = document.createElement("canvas");
                                    let context = canvas.getContext("2d");

                                    let startbutton = document.createElement("button");
                                    startbutton.textContent = "Foto aufnehmen";
                                    startbutton.classList.add('btn', 'btn-primary');
                                    startbutton.style.position = "fixed";
                                    startbutton.style.top = "70%";
                                    startbutton.style.left = "50%";
                                    startbutton.style.transform = "translateX(-50%)";

                                    let stopbutton = document.createElement("button");
                                    stopbutton.innerHTML = '<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" class="bi bi-x" viewBox="0 0 16 16"> <path d="M4.646 4.646a.5.5 0 0 1 .708 0L8 7.293l2.646-2.647a.5.5 0 0 1 .708.708L8.707 8l2.647 2.646a.5.5 0 0 1-.708.708L8 8.707l-2.646 2.647a.5.5 0 0 1-.708-.708L7.293 8 4.646 5.354a.5.5 0 0 1 0-.708"/> </svg>';
                                    stopbutton.classList.add('btn', 'btn-primary');
                                    stopbutton.style.position = "fixed";
                                    stopbutton.style.top = "26%";
                                    stopbutton.style.left = "65%";
                                    stopbutton.style.transform = "translateX(-50%)";

                                    startbutton.addEventListener("click", function () {
                                        context.drawImage(video, 0, 0, canvas.width, canvas.height);
                                        let imgData = canvas.toDataURL("image/png");

                                        console.log(imgData);

                                        // Foto als Data URL (base64) und weiterverarbeiten

                                        let img = document.createElement("img");
                                        img.src = imgData; // Setze die Data URL des Fotos als Wert für das src-Attribut

                                        let kategorieName = id.split(".")[0];
                                        let schrittName = id.split(".")[1];

                                        if (!vm.pruefung.pruefung[kategorieName]) vm.pruefung.pruefung[kategorieName] = {};
                                        if (!vm.pruefung.pruefung[kategorieName][schrittName]) vm.pruefung.pruefung[kategorieName][schrittName] = {};
                                        vm.pruefung.pruefung[kategorieName][schrittName]["value"] = imgData;

                                        // Füge das <img>-Element dem content hinzu
                                        content.innerHTML = ''; // Leere den Inhalt von content, um sicherzustellen, dass das vorherige Foto entfernt wird
                                        content.appendChild(img);

                                        element.classList.add('table-success');
                                    });

                                    stopbutton.addEventListener("click", function () {
                                        let video = document.getElementById("fotoaufnahme");

                                        let stream = video.srcObject;
                                        let tracks = stream.getTracks();
                                        tracks.forEach(function (track) {
                                            track.stop();
                                        });

                                        video.srcObject = null;
                                        video.remove();
                                        //video = null;

                                        startbutton.remove();
                                        stopbutton.remove();
                                    });

                                    document.body.appendChild(video);
                                    document.body.appendChild(startbutton);
                                    document.body.appendChild(stopbutton);
                                })
                                .catch(function (error) {
                                    // Fehler beim Öffnen der Kamera
                                    console.error('Fehler beim Öffnen der Kamera:', error);
                                });
                        });
                        
                        /* add.addEventListener("change", element.classList.add('table-success')); */
                        
                            let foto = document.createElement("input");
                            foto.type = "image";
                            foto.id = "' + id + '-schritt-div-content";

                            foto.addEventListener("input", function()
                                {if (foto = empty) 
                                    {element.classList.add('table-success');} 
                                else {element.classList.remove('table-success');}

                            // Erstelle ein <img>-Element, um das gespeicherte Foto anzuzeigen
                            let img = document.createElement("img");
                            img.src = imgData; // Setze die Data URL des Fotos als Wert für das src-Attribut

                            // Füge das <img>-Element dem content hinzu
                            content.appendChild(img);
                        });

                    let comment = document.createElement("input");
                    comment.type = "text";
                    comment.id = "' + id + '-schritt-div-comment";
                    
                    //Überprüfung, ob Inhalt schon existiert und dann einfügen
                    if (this.check.model[firstID][displayName]["Kommentar"]) text.value = this.check.model[firstID][displayName]["Kommentar"];

                    comment.addEventListener("input", () =>
                    {if (comment.value.trim() === "") 
                        {;} 
                    else 
                    {
                        //element.classList.add('table-success');
                        this.userInput = comment.value;
                        this.check.model[firstID][displayName]["Kommentar"]=this.userInput;
                    }
                    });

                    comm.appendChild(comment);

                        view.innerHTML = '<button class="btn btn-lg menu-button" type="button" style="padding: 0;">  <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" class="bi bi-x-lg" viewBox="0 0 16 16"> <path d="M2.146 2.854a.5.5 0 1 1 .708-.708L8 7.293l5.146-5.147a.5.5 0 0 1 .708.708L8.707 8l5.147 5.146a.5.5 0 0 1-.708.708L8 8.707l-5.146 5.147a.5.5 0 0 1-.708-.708L7.293 8z"/> </svg></button> ';

                        view.addEventListener("click", function()
                        {
                            content.innerHTML = "";
                            element.classList.remove('table-success');
                        });
                        break;
                    }
                case "Text":
                {
                    add.addEventListener("click", function() { vm.startScan(replacementString)});
                    
                    //add.innerHTML = '<button class="btn btn-outline-secondary btn-lg menu-button me-md-2" type="button"><svg xmlns="http://www.w3.org/2000/svg" width="25" height="25" fill="currentColor" class="bi bi-pen" viewBox="0 0 16 16"> <path d="m13.498.795.149-.149a1.207 1.207 0 1 1 1.707 1.708l-.149.148a1.5 1.5 0 0 1-.059 2.059L4.854 14.854a.5.5 0 0 1-.233.131l-4 1a.5.5 0 0 1-.606-.606l1-4a.5.5 0 0 1 .131-.232l9.642-9.642a.5.5 0 0 0-.642.056L6.854 4.854a.5.5 0 1 1-.708-.708L9.44.854A1.5 1.5 0 0 1 11.5.796a1.5 1.5 0 0 1 1.998-.001m-.644.766a.5.5 0 0 0-.707 0L1.95 11.756l-.764 3.057 3.057-.764L14.44 3.854a.5.5 0 0 0 0-.708z"/> </svg></button> ' ;
                    
                    let text = document.createElement("input");
                    
                    text.type = "text";
                    //text.id = "' + id + '-schritt-div-content";
                    text.id = `${id}-schritt-div-content`;
                    const existingDescription = this.check.model[firstID][displayName]["Beschreibung"];
                    if (existingDescription) {
                        text.value = existingDescription;
                        element.classList.add('table-success');
                    }   


                    console.log("textfeld")
                    console.log(this.check.model[firstID][displayName]["Beschreibung"])

                    text.addEventListener("input", () =>
                        {if (text.value.trim() === "") 
                            {element.classList.remove('table-success');} 
                        else 
                        {
                            element.classList.add('table-success');
                            this.userInput = text.value;
                            this.check.model[firstID][displayName]["Beschreibung"]=this.userInput
                        }
                    });

                    view.innerHTML = '<button class="btn btn-lg menu-button" type="button">  <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" class="bi bi-x-lg" viewBox="0 0 16 16"> <path d="M2.146 2.854a.5.5 0 1 1 .708-.708L8 7.293l5.146-5.147a.5.5 0 0 1 .708.708L8.707 8l5.147 5.146a.5.5 0 0 1-.708.708L8 8.707l-5.146 5.147a.5.5 0 0 1-.708-.708L7.293 8z"/> </svg></button> ';
                    
                    view.addEventListener("click", function()
                                {
                                    text.value = "";
                                    element.classList.remove('table-success');
                                });

                    content.appendChild(text);
                    break;
                }
                case "Checkbox":
                {        /* content.innerHTML = '<input type="checkbox" st id="' + id + '-schritt-div-content" name="checkbox" />'; */
                        content.innerHTML = '<span id="' + id + '-schritt-div-content">' + schritt.Beschreibung + " </span>";
                        let checkbox = document.createElement("input");
                        checkbox.type = "checkbox";
                        checkbox.style.transform ="scale(2)";
                        checkbox.id = "' + id + '-schritt-div-checkbox";

                        //Überprüfung, ob Checkbox schon ausgewählt und dann einfügen
                        if (this.check.model[firstID][displayName]["checkbox"]) {checkbox.checked = this.check.model[firstID][displayName]["checkbox"];}                        
                        
                        checkbox.addEventListener("change", () => {
                            if (checkbox.checked) {
                                element.classList.add('table-success');
                                console.log("checkbox checked");
                                this.check.model[firstID][displayName]["checkbox"] = true;
                            } else {
                                element.classList.remove('table-success');
                                this.check.model[firstID][displayName]["checkbox"] = false;
                                }
                            });

                        let comment = document.createElement("input");
                        comment.type = "text";
                        comment.id = "' + id + '-schritt-div-comment"; // hier der Kommentarbereich von Schritt 5 

                        //Überprüfung, ob Inhalt schon existiert und dann einfügen
                        if (this.check.model[firstID][displayName]["Kommentar"]) text.value = this.check.model[firstID][displayName]["Kommentar"];

                        comment.addEventListener("input", () =>
                        {if (comment.value.trim() === "") 
                            {;} 
                        else 
                        {
                            //element.classList.add('table-success');
                            this.userInput = comment.value;
                            this.check.model[firstID][displayName]["Kommentar"]=this.userInput;
                        }
                    });

                        comm.appendChild(comment);

                        view.appendChild(checkbox);
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
                    
                    view.innerHTML = '<button class="btn btn-lg menu-button" type="button" style="padding: 0;">  <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" class="bi bi-x-lg" viewBox="0 0 16 16"> <path d="M2.146 2.854a.5.5 0 1 1 .708-.708L8 7.293l5.146-5.147a.5.5 0 0 1 .708.708L8.707 8l5.147 5.146a.5.5 0 0 1-.708.708L8 8.707l-5.146 5.147a.5.5 0 0 1-.708-.708L7.293 8z"/> </svg></button> ';    

                    let comment = document.createElement("input");
                    comment.type = "text";
                    comment.id = "' + id + '-schritt-div-comment";
                    
                    //Überprüfung, ob Inhalt schon existiert und dann einfügen
                    if (this.check.model[firstID][displayName]["Kommentar"]) text.value = this.check.model[firstID][displayName]["Kommentar"];

                    comment.addEventListener("input", () =>
                    {if (comment.value.trim() === "") 
                        {;} 
                    else 
                    {
                        //element.classList.add('table-success');
                        this.userInput = comment.value;
                        this.check.model[firstID][displayName]["Kommentar"]=this.userInput;
                    }
                    });

                    comm.appendChild(comment);
                    
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
            element.appendChild(edit);
            element.appendChild(add);
            element.appendChild(view);
            element.appendChild(comm);
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
                console.log("schirtt-3")
                console.log(schritt)
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
            console.log("FunktionNeuePruefung")
            this.pruef_status = "neue_pruefung";
            console.log(this.pruef_status)
            this.nextStep();
            this.$refs.step1.classList.remove('active');
            this.$refs.step2a.classList.add('active');

        },
        FunktionPruefungFortsetzen()
        {
            //Parameter zur Unterscheidung beim bauen der HTML welches this.check.model verwendet wird - beim Weiter Button
            this.pruef_status="zw_pruefung";

            // Übersichtsliste der zwischengespicherten Prüfungen bekommen
            axios.get('http://localhost:4000/Zwischenspeicherverzeichnis_List')
            .then((response) => { 
                console.log('http://localhost:4000/Zwischenspeicherverzeichnis_List')            
                this.check.gespeicherte_pruefungen = response.data;

                for (const [key, value] of Object.entries(this.check.gespeicherte_pruefungen))
                {
                    this.check.gespeicherte_pruefungen_lesbar[key] = "";
                    for (const pruefplanname in value)
                    {
                        this.check.gespeicherte_pruefungen_lesbar[key] += value[pruefplanname] + ", "
                    }
                    this.check.gespeicherte_pruefungen_lesbar[key] = this.check.gespeicherte_pruefungen_lesbar[key].substring(0, this.check.gespeicherte_pruefungen_lesbar[key].length - 2);
                }
    
            })
            .catch(function (error) {
                console.log('error mess');
                console.error(error);
            });
 
            this.nextStep();
            this.$refs.step1.classList.remove('active');
            this.$refs.step2b.classList.add('active');
        },
        modellAus()
        {
            var attachmentPoint = document.getElementById('checkInjectionPoint');

            let table = document.createElement("table");

            var vm = this;

            for (const [key, value] of Object.entries(this.check.model))
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
        async nextStep() {
            if (this.step == 1) {
                
                // Liste der Zwischengespeicherten Tests aktualisieren bevor diese neu zugesandt werden 
                // (soll auch Funktionieren, wenn direkt nach Zwischenspiechern zurück auf Home-Screen und dann dort auf neue Liste zugreifen)
                axios.get('http://localhost:4000/createZwischenspeicherverzeichnis_List')
                .then((response) => {

                    console.log("createZwischenspeicherverzeichnis_List");

                    this.check.pruefplaene = response.data;
                    for (const [key, value] of Object.entries(this.check.pruefplaene))
                    {
                        this.check.pruefplane_lesbar[key] = "";
                        for (const pruefplanname in value)
                        {
                            this.check.pruefplane_lesbar[key] += value[pruefplanname] + ", "
                        }
                        this.check.pruefplane_lesbar[key] = this.check.pruefplane_lesbar[key].substring(0, this.check.pruefplane_lesbar[key].length - 2);
                    }

                })
                .catch(function (error) {
                    console.log('error mess');
                    console.error(error);
                });

                this.step++;
            } else if (this.step == 2) {
                console.log("Step 2 - Prüfpläne")
                console.log(this.pruef_status)
                if (this.pruefplanId == "" && this.gespeicherte_pruefungId == "") {
                    return;
                } 

                // Untrescheidung ob Neue PRüfung oder aus gespeicherter Liste
                if (this.pruef_status == "neue_pruefung"){

                    await axios.get('http://localhost:4000/pruefplaene/' + this.$refs.auswahlPruefplan.value)
                        .then((response) => {
                            if (response.status == 200) {
                                console.log("http://localhost:4000/pruefplaene/")
                                this.$refs.step2a.classList.remove('active');
                                this.$refs.step3.classList.add('active');
                                this.step++;

                                this.check.model = response.data;
                                this.modellVorbereiten();

                                this.eingangsinformationenEinhaengen();
                                let element = document.getElementById("Eingangsinformationen-kategorie-div");
                                element.style.display = "block";
                            } else {
                                alert("Fehler beim Laden des Prüfplans, HTTP-Statuscode: " + response.status);
                            }
                        })
                        .catch((error) => {
                            console.log(error);
                            alert("Fehler beim Laden des Prüfplans");
                        });
                }
                else if (this.pruef_status == "zw_pruefung"){

                    await axios.get('http://localhost:4000/pruefungen/speichern/' + this.$refs.auswahlGespeichertePruefung.value) //ändern
                    .then((response) => {       
                            console.log("http://localhost:4000/pruefungen/speichern/")            
                            if (response.status == 200) {

                                this.$refs.step2b.classList.remove('active');
                                this.$refs.step3.classList.add('active');

                                // Aus string der Json Dateinamen jeweils nur den Anfangstext nehmen und rest abschneiden ab "_"
                                let name_pruefung_long = this.$refs.auswahlGespeichertePruefung.value
                                let underscoreIndex = name_pruefung_long.indexOf('_');
                                let name_pruefung = underscoreIndex !== -1 ?
                                name_pruefung_long.substring(0, underscoreIndex) : name_pruefung_long;
                                this.$refs.auswahlPruefplan.value = name_pruefung

                                console.log("this.$refs.auswahlPruefplan.value")
                                console.log(this.$refs.auswahlPruefplan.value)
                                this.step++;
                                //this.step++;

                                this.check.model = response.data;
                                this.modellVorbereiten();

                                this.eingangsinformationenEinhaengen();
                                let element = document.getElementById("Eingangsinformationen-kategorie-div");
                                element.style.display = "block";


                            } else {
                                alert("Fehler beim Laden des Prüfplans, HTTP-Statuscode: " + response.status);
                            }
                        })
                        .catch((error) => {
                            console.log(error);
                            alert("Fehler beim Laden des Prüfplans");
                        });
                    }       
            } else if (this.step == 3) {
                this.$refs.step3.classList.remove('active');
                this.$refs.step4.classList.add('active');
                this.step++;
            } else if (this.step == 4) {
                this.$refs.step4.classList.remove('active');
                this.$refs.step5.classList.add('active');
                //alert(JSON.stringify(this.check.model, null, 2));
                this.step++;

                this.kategorienEinhaengen();

                for (const [kategorieName, kategorieInhalt] of Object.entries(this.pruefung.pruefung)) {
                    let element = document.getElementById(kategorieName + "-kategorie-div");
                    let aktiveKategorie = this.darstellung.aktiveKategorie;
                    if (kategorieName == this.darstellung.aktiveKategorie) {
                        //element.classList.add("active");
                        //console.log(kategorieName)
                        //console.log("add active")
                        element.style.display = "block";
                    } else {
                        //element.classList.remove("active");
                        //console.log(kategorieName)
                        //console.log("remove active")
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
                //this.$refs.step3.classList.remove('active');
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
            // Preference Gerät Name - noch ändern zum Name des Prüfauftrags (automatisch auslesen)
            var name = " PreferenceGerätName";
            
            console.log("submit");
            axios.post('http://localhost:4000/pruefungen/senden?name=' + name ,this.check.model)
                .then((response) => 
                {
                    console.log("Submission sucess.")
                    console.log(response)
                })
            .catch(function (error) {
                console.log('error mess');
                console.error(error);
            });
            console.log("Alert");
            this.showAlert = true;
            setTimeout(() => {
                this.showAlert = false;
            }, 3000);

            if (!this.checkAlle)
            {
                console.log("submit geht nicht")
            }
            setTimeout(() => {
                this.step = 1;
                this.model = {};

                this.$refs.step5.classList.remove('active');
                this.$refs.step4.classList.remove('active');
                this.$refs.step1.classList.add('active');
            }, 3000);

            //this.step = 1;
            //this.modellVorbereiten();
        },
        checkAlle() {
            console.log("checkAlle");

            this.kategorienUeberpruefen();

            let res = true;

            for (const [key, value] of Object.entries(this.check.model))
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
            console.log("vorbereitet");
            console.log(vorbereitet);

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

            //this.pruefung.aktiveKategorie = kategorieName;
            //console.log(this.pruefung.aktiveKategorie);
            this.darstellung.aktiveKategorie = kategorieName;
            console.log(this.darstellung.aktiveKategorie);
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
            //const url = 'http://localhost:4000/pruefungen/senden';
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

        // Funktion zum Formatieren der aktuellen Datum und Uhrzeit
        getCurrentDateTime() {
            const now = new Date();
            const Stunde = String(now.getHours()).padStart(2, '0');
            const Minute = String(now.getMinutes()).padStart(2, '0');
            const Tag = String(now.getDate()).padStart(2, '0');
            const Monat = String(now.getMonth() + 1).padStart(2, '0'); // Monate sind 0-basiert
            const Jahr = now.getFullYear();

            return `${Stunde}${Minute}_${Tag}${Monat}${Jahr}`;
        },

        pruefungZwischenspeichern() {
            // v1 ohne zip nur direkt model json
             // Preference Gerät Name
            //var name = "Pruefplan_1123_13082024"; // Name anpassen zu Datum und Uhrzei und Prüfname?
            //var name = "Pruefplan_" + this.getCurrentDateTime();
            console.log("this.$refs.auswahlPruefplan.value - 2")
            console.log(this.$refs.auswahlPruefplan.value)
            var name = this.$refs.auswahlPruefplan.value + "_" + this.getCurrentDateTime();

            
            console.log("pruefungZwischenspeichern()");

            axios.post('http://localhost:4000/pruefungen/speichern?name=' + name ,this.check.model) // hier Endung mit ? anpassen? 
                .then((response) => 
                {
                    console.log("Submission sucess.")
                    console.log(response)
                })
            .catch(function (error) {
                console.log('error mess');
                console.error(error);
            });
            console.log("Alert");
            this.showAlert = true;
            setTimeout(() => {
                this.showAlert = false;
            }, 3000);

            if (!this.checkAlle)
            {
                console.log("submit geht nicht")
            }
            setTimeout(() => {
                this.step = 1;
                this.model = {};

                this.$refs.step5.classList.remove('active');
                this.$refs.step4.classList.remove('active');
                this.$refs.step1.classList.add('active');
            }, 3000);

            //this.step = 1;
            //this.modellVorbereiten();           

            // hier die Version von david mit Zip
            let zip = new JSZip();
            let elements = document.querySelectorAll("[type='image']");
            console.log("pruefungZwischenspeichern")
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
                    uploadFile("URL-Link", content);
                });
        }
    },
    // Aufbau planen - wo welche Daten herausnehem naus Server und wie belegen
    data() {
        return {
            //object Einstellung von hier lesen
            //const url = 'http://localhost:4000/pruefungen/senden';
            //auch andere Server Axios links 
            auswahlPruefplan: "",
            showAlert: false,
            proxyData: null,
            step: 1,
            pruef_status: "",
            nextStepMoeglich: true,
            scanNichtVerfuegbar: false,
            userInput: "",
            check: {
                //model: Modell,
                model: {},
                //pruefplaene: Pruefplaeneverzeichnis,
                
                pruefplaene: {},
                pruefplane_lesbar: {},
                pruefplanId: "",
                create_gespeicherte_preufungen: {},
                gespeicherte_pruefungen: {
                    // pruefung1: {},
                    // pruefung2: {},
                    // pruefung3: {},
                    // pruefung4: {},
                    // pruefung5: {},
                    // pruefung6: {}
                },
                gespeicherte_pruefungen_lesbar:{},
                gespeicherte_pruefungId: ""
            },
            pruefung: {
                pruefplan: {},
                pruefplan_lesbar: {}, // abgeschnitten der Endung damit lesbar
                pruefplanId: "",
                pruefung: null, // hier die Infos rein und auch Kategorien sind hier drin
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
                    Schritt41: {
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
    <div class="menubar mb-5">
        <a class="navbar-brand" href="step1_start">
            <img src="/src/assets/FinalesLogo_2022-10_rgb_aufgeräumt_gross.png" alt="Logo" width="120" class="d-inline-block align-text-top">
        </a>
        <h1 class="me-md-4">SAM-KI-Check</h1>
        <h2 class="me-md-4">Schritt {{step}}</h2>
        
        <button class="btn btn-secondary btn-lg menu-button me-md-2" type="button" id="dropdownMenuButton1" data-bs-toggle="dropdown" aria-expanded="false" align="right">
            <svg xmlns="http://www.w3.org/2000/svg" width="30" height="30" fill="white" class="bi bi-list" viewBox="0 0 16 16">
                    <path fill-rule="evenodd" d="M2.5 12a.5.5 0 0 1 .5-.5h10a.5.5 0 0 1 0 1H3a.5.5 0 0 1-.5-.5m0-4a.5.5 0 0 1 .5-.5h10a.5.5 0 0 1 0 1H3a.5.5 0 0 1-.5-.5m0-4a.5.5 0 0 1 .5-.5h10a.5.5 0 0 1 0 1H3a.5.5 0 0 1-.5-.5"/>
            </svg>
        </button>
        <ul class="dropdown-menu" v-if="step==1" aria-labelledby="dropdownMenuButton1">
            <li><a class="dropdown-item text-center"  v-if="step==1" @click="setSettings">Einstellung</a></li>
        </ul>
        

        <button class="btn btn-outline-secondary btn-lg btn-light" type="button" @click="scan_startScan">Senden</button>
        
    </div>

    <!-- Einstellung -->
    <div id="step1b_einstellung" class="step" ref="step1b">
        <div >
            <div class="mb-5 h1 text-center">Einstellung</div>
            <div class="text-center">
                <table class="table1 table-responsive" style="max-height: 600px; overflow: auto; margin-left:200px;">
                    <thead>
                        <tr>
                        <th scope="col">Parameter</th>
                        <th scope="col"></th>
                        <th scope="col"></th>

                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                        <th scope="row">1</th>
                        <td>Preference Gerät Name</td>
                        <td>Eingabe...</td>
                        </tr>
                        <tr>
                        <th scope="row">2</th>
                        <td>IP</td>
                        <td>Eingabe...</td>
                        </tr>
                    </tbody>
                </table>
        </div>
        </div>
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
                <select class="form-select form-select-lg mb-3" size="12" v-model="check.pruefplanId" id="selectionPruefplan" ref="auswahlPruefplan">
                    <option v-for="(value, key) in check.pruefplane_lesbar" :key="key" :value="value">
                        <!--{{ key }} für {{ value }} -->
                        {{ value }}
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
                <select class="form-select form-select-lg mb-3" size="12" v-model="check.gespeicherte_pruefungenId" id="selectionGespeichertePruefung" ref="auswahlGespeichertePruefung">
                    <option v-for="(value, key) in check.gespeicherte_pruefungen_lesbar" :key="key" :value="value">
                        {{ value }}
                    </option>
                </select>
            </div>
        </div>
    </div>

    <!-- Prüfauftrag -->
    <div id="step3_auftragsinfos" class="step" ref="step3">
        <div >
            <div class="mb-5 h1 text-center">Prüfauftrag</div>

            <table class="table table-responsive justify-content-md-center md-3" style="max-height: 500px; overflow: auto; margin-left: auto; margin-right: auto;" id="injectionPointAuftragsinfos">
            </table>
        </div>
    </div>

    <!-- Übersicht des ausgewählten Prüfplans -->
    <div id="step4_uebersicht" class="step" ref="step4">
        <div class="container h1">
            <div class="mb-5 text-center">Übersicht</div>
                
            <div class="container">
                <div class="row">
                    <div class="col-9">
                        <div class="row">
                            <!-- <div v-for="(value, key, index) in pruefung.pruefung" :key="key" :value="value" class="col-4 mb-3"> -->
                            <div v-for="(value, key, index) in check.model" :key="key" :value="value" class="col-4 mb-3">
                                <button v-if="key !== 'Eingangsinformationen'" 
                                        :class="[ 'btn', 'btn-lg', 'w-100', 
                                            { 'btn-success': value.erfuellteSchritte === value.anzahlSchritte, 
                                            'btn-secondary': value.erfuellteSchritte !== value.anzahlSchritte } ]"
                                        style="height: 10rem;" 
                                    @click="kategorieStarten(key)">
                                    <span class="title" style="font-weight:bold; ">{{ key }}</span>
                                    <div class="progress" role="progressbar" aria-label="Basic example" aria-valuenow="{{ value.erfuellteSchritte }}/{{ value.anzahlSchritte }}" aria-valuemin="0" aria-valuemax="1">
                                        <div class="progress-bar" :style="{ width: (value.erfuellteSchritte / value.anzahlSchritte * 100) + '%' }"> 
                                            {{ value.erfuellteSchritte }}/{{ value.anzahlSchritte }} 
                                        </div>
                                    </div>
                                </button>
                            </div>
                        </div>
                    </div>

                    
                    <div class="col-3 card text-bg-light h3">
                    <!-- erstmal Dummy-Daten -->
                        <div class="row" style="padding: 10px; width:" v-for="(value, key, index) in check.model.Eingangsinformationen" :key="key" :value="value">
                            <div class="col" :id="'ueberblick.eingangsinformationen.' + key" v-if="index < (Object.keys(check.model.Eingangsinformationen).length - 2)">
                                <span style="font-size: 14px;">{{ key+": " }} {{ value.Beschreibung }}</span>
                            </div>
                        </div>
                    </div>

                </div>
            </div>
        </div>
    </div>

    <div id="step5_pruefung" class="step" ref="step5">
        <div class="mb-5 h1 text-center" v-if="darstellung.aktiveKategorie">
            {{ darstellung.aktiveKategorie }}</div>
        <div class="container row row-col-3 justify-content-md-center" style="display: flex; margin-left: auto; margin-right: auto; font-weight: bold; padding:0; padding-bottom: 10px;">
                    <div class="col-md-2">Prüf-Nr.</div>
                    <div class="col">Beschreibung Prüfschritt</div>
                    <div class="col"> </div>
                    <div class="col" style="text-align: right;">Aktion</div>
                    <div class="col"> </div>
                    <div class="col">Kommentar Bearbeiter</div>
            </div>
        <div class="table table-responsive" style="max-height: 350px; overflow: auto;" id="injectionPointKategorien">

        </div>
    </div>

    <!-- Weiter und Zurück Buttons -->
    <div class="m-3">
      <button type="button" v-if="step==1" disabled class="btn btn-outline-secondary btn-lg step-button-left-inactive fs-1 position-absolute bottom-0 start-0">Zurück</button>
      <button type="button" @click="prevStep" v-if="step!=1" class="btn btn-outline-secondary btn-bg-white btn-lg step-button-left fs-1 position-absolute bottom-0 start-0">Zurück</button>

      <button type="button" v-if="!nextStepMoeglich" disabled class="btn btn-secondary btn-lg step-button-right-inactive fs-1 position-absolute bottom-0 " style="right: 300px;">Zwischenspeichern</button>
      <button type="button" @click="pruefungZwischenspeichern" v-if="step<5 && step!=1 && nextStepMoeglich" class="btn btn-secondary btn-lg step-button-left fs-1 position-absolute bottom-0 " style="right: 300px;">Zwischenspeichern</button>

      <button type="button" v-if="!nextStepMoeglich" disabled class="btn btn-secondary btn-lg step-button-right-inactive fs-1 position-absolute bottom-0 end-0 translate-middle-x">Weiter</button>
      <button type="button" @click="nextStep" v-if="step<4 && step!=1 && nextStepMoeglich" class="btn btn-secondary btn-lg step-button-right fs-1 position-absolute bottom-0 end-0 translate-middle-x">Weiter</button>
      <!--
      <button type="button" @click="submit" v-if="step==4" class="btn btn-primary btn-lg step-button-right fs-1 position-absolute bottom-0 end-0 translate-middle-x">Senden</button> 
      -->
      <!-- <button type="button" @click="submit" v-if="step==4" class="btn btn-lg btn-primary btn-lg step-button-right fs-1 position-absolute bottom-0 end-0 translate-middle-x" data-bs-toggle="popover" data-bs-placement="left" data-bs-title="Popover title" data-bs-content="And here's some amazing content. It's very engaging. Right?">Senden</button> -->
      <button type="button" @click="submit" v-if="step==4" class="btn btn-primary btn-lg step-button-right fs-1 position-absolute bottom-0 end-0 translate-middle-x">Senden</button> 
      <!-- <button type="button" v-if="step==4" class="btn btn-lg btn-primary btn-lg step-button-right fs-1 position-absolute bottom-0 end-0 translate-middle-x" data-bs-toggle="popover" title="Popover title" data-bs-content="Here's some amazing content.">Senden</button> -->
    </div>

    <div v-if="showAlert" class="alert alert-success custom-alert">
        <strong>Erfolgreich gesendet!</strong> Sie werden zurück zur Startseite geleitet.
      </div>
      
     



   <!--  <div id="checkInjectionPoint">hello</div> -->
   <!-- <button @click="pruefungZwischenspeichern" class="btn btn-lg">Test</button> -->
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

/*
.table {
    display: flex;
    width: 600px;
    margin: 0 auto;
    background-color: #fbfcfc;
}
*/
.custom-alert {
    width: 50%;       /* Breite des Alerts auf 50% des Bildschirms setzen */
    margin: 0 auto;   /* Zentriert den Alert horizontal */
    position: fixed;  /* Fixiert den Alert über anderen Inhalten */
    bottom: 200px;        /* Abstand vom oberen Rand */
    left: 0;
    right: 0;
    z-index: 9999; 
    text-align: center;   /* Stellt sicher, dass der Alert über anderen Elementen liegt */
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
