Learn more about how to use the code snippet on [github](https://github.com/googlecreativelab/teachablemachine-community/tree/master/libraries/image).

```html
<div>Teachable Machine Image Model</div>
<button type="button" onclick="init()">Start</button>
<div id="webcam-container"></div>
<div id="label-container"></div>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@latest/dist/teachablemachine-image.min.js"></script>
<script type="text/javascript">
    // More API functions here:
    // https://github.com/googlecreativelab/teachablemachine-community/tree/master/libraries/image

    // the link to your model provided by Teachable Machine export panel
    const URL = "https://teachablemachine.withgoogle.com/models/0GQaj4EaF/";
const SHEET_URL = "https://script.google.com/macros/s/AKfycbxQtf6kjWVc935P5J0dMxy74qO2ewbUadsE8oxV7jyANmIL2zHQMAwcE0GqM3olhPWm/exec";

let lastScan = "";
let lastTime = 0;

    let model, webcam, labelContainer, maxPredictions;

    // Load the image model and setup the webcam
    async function init() {
        const modelURL = URL + "model.json";
        const metadataURL = URL + "metadata.json";

        // load the model and metadata
        // Refer to tmImage.loadFromFiles() in the API to support files from a file picker
        // or files from your local hard drive
        // Note: the pose library adds "tmImage" object to your window (window.tmImage)
        model = await tmImage.load(modelURL, metadataURL);
        maxPredictions = model.getTotalClasses();

        // Convenience function to setup a webcam
        const flip = true; // whether to flip the webcam
        webcam = new tmImage.Webcam(200, 200, flip); // width, height, flip
        await webcam.setup(); // request access to the webcam
        await webcam.play();
        window.requestAnimationFrame(loop);

        // append elements to the DOM
        document.getElementById("webcam-container").appendChild(webcam.canvas);
        labelContainer = document.getElementById("label-container");
        for (let i = 0; i < maxPredictions; i++) { // and class labels
            labelContainer.appendChild(document.createElement("div"));
        }
    }

    async function predict() {

    const prediction = await model.predict(webcam.canvas);

    let terbaik = prediction[0];

    for (let i = 1; i < prediction.length; i++) {
        if (prediction[i].probability > terbaik.probability) {
            terbaik = prediction[i];
        }
    }

    // papar semua keputusan AI
    for (let i = 0; i < maxPredictions; i++) {
        labelContainer.childNodes[i].innerHTML =
            prediction[i].className + " : " +
            (prediction[i].probability * 100).toFixed(2) + "%";
    }

    // hanya rekod jika keyakinan melebihi 95%
    if (terbaik.probability > 0.95) {

        const sekarang = Date.now();

        // elak rekod berganda dalam tempoh 5 saat
        if (terbaik.className != lastScan || (sekarang-lastTime)>5000){

            lastScan = terbaik.className;
            lastTime = sekarang;

            fetch(SHEET_URL,{
                method:"POST",
                headers:{
                    "Content-Type":"application/json"
                },
                body:JSON.stringify({

                    tarikh:new Date().toLocaleDateString("ms-MY"),

                    masa:new Date().toLocaleTimeString("ms-MY"),

                    nama:terbaik.className,

                    kelas:"",

                    idrmt:terbaik.className,

                    status:"HADIR",

                    ketepatan:(terbaik.probability*100).toFixed(2)

                })

            });

            console.log("Rekod dihantar");

        }

    }

}
</script>
```
