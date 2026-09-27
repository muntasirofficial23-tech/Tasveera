{
  "name": "prompt2video-backend",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "dotenv": "^16.4.7",
    "express": "^4.21.2"
  }
}require("dotenv").config();

const express = require("express");
const cors = require("cors");
const path = require("path");

const app = express();

app.use(cors());
app.use(express.json({ limit: "1mb" }));

app.use(express.static(path.join(__dirname, "public")));

const jobs = new Map();

app.post("/api/generate", async (req, res) => {

    const {
        prompt,
        ratio = "16:9",
        quality = "1080p",
        duration = 5
    } = req.body || {};

    if (!prompt || !prompt.trim()) {
        return res.status(400).json({
            error: "Prompt is required"
        });
    }

    const jobId =
        Date.now().toString(36) +
        Math.random().toString(36).slice(2, 7);

    jobs.set(jobId, {
        id: jobId,
        status: "queued",
        progress: 0,
        prompt,
        ratio,
        quality,
        duration,
        videoUrl: null
    });

    /*
      DEMO GENERATION

      Yahan actual AI video-generation
      provider ka API connect kiya jayega.
    */

    simulate(jobId);

    res.json({
        jobId
    });
});


app.get("/api/jobs/:id", (req, res) => {

    const job = jobs.get(req.params.id);

    if (!job) {
        return res.status(404).json({
            error: "Job not found"
        });
    }

    res.json(job);
});


function simulate(id) {

    const timer = setInterval(() => {

        const job = jobs.get(id);

        if (!job) {
            clearInterval(timer);
            return;
        }

        job.progress =
            Math.min(job.progress + 10, 100);

        if (job.progress >= 100) {

            job.status = "completed";

            /*
              Actual provider se video URL
              yahan set hoga.
            */

            job.videoUrl = null;

            clearInterval(timer);

        } else {

            job.status = "processing";

        }

    }, 700);
}


app.get("*", (req, res) => {

    res.sendFile(
        path.join(
            __dirname,
            "public",
            "index.html"
        )
    );

});


const PORT =
    process.env.PORT || 3000;


app.listen(PORT, () => {

    console.log(
        `Prompt2Video running on http://localhost:${PORT}`
    );

});<!doctype html>

<html lang="en">

<head>

<meta charset="utf-8">

<meta
name="viewport"
content="width=device-width,initial-scale=1">

<title>Prompt2Video AI</title>

<style>

* {
    box-sizing: border-box;
}

body {

    margin: 0;

    background: #090b12;

    color: white;

    font-family:
        Arial,
        sans-serif;
}

main {

    max-width: 900px;

    margin: auto;

    padding: 30px 18px;
}

h1 {

    font-size: 42px;

    text-align: center;

    background:
        linear-gradient(
            90deg,
            #7c5cff,
            #00d4ff
        );

    color: transparent;

    background-clip: text;
}

.card {

    background: #111522;

    border:
        1px solid #282f40;

    border-radius: 18px;

    padding: 20px;
}

textarea {

    width: 100%;

    min-height: 160px;

    background: #080b12;

    color: white;

    border:
        1px solid #343b4e;

    border-radius: 12px;

    padding: 15px;

    font-size: 16px;

    resize: vertical;
}

.controls {

    display: flex;

    gap: 10px;

    flex-wrap: wrap;

    margin-top: 14px;
}

select,
button {

    padding: 12px;

    border-radius: 10px;

    border:
        1px solid #343b4e;
}

select {

    background: #0c1019;

    color: white;
}

button {

    background:
        linear-gradient(
            90deg,
            #7657ff,
            #00bfe8
        );

    color: white;

    border: 0;

    font-weight: bold;

    cursor: pointer;
}

button:hover {

    opacity: .9;
}

#box {

    margin-top: 20px;

    display: none;
}

.bar {

    height: 9px;

    background: #242a38;

    border-radius: 8px;

    overflow: hidden;
}

.fill {

    height: 100%;

    width: 0;

    background:
        linear-gradient(
            90deg,
            #7657ff,
            #00d9ff
        );
}

video {

    width: 100%;

    margin-top: 15px;

    border-radius: 12px;
}

</style>

</head>


<body>


<main>

<h1>
Prompt2Video AI
</h1>


<div class="card">


<textarea
id="prompt"
placeholder="Describe the video you want..."
></textarea>


<div class="controls">


<select id="ratio">

<option value="16:9">
16:9 Landscape
</option>

<option value="9:16">
9:16 Portrait
</option>

<option value="1:1">
1:1 Square
</option>

</select>


<select id="quality">

<option value="1080p">
1080p
</option>

<option value="720p">
720p
</option>

</select>


<select id="duration">

<option value="5">
5 seconds
</option>

<option value="10">
10 seconds
</option>

<option value="15">
15 seconds
</option>

</select>


<button onclick="generate()">
Generate Video
</button>


</div>


<div id="box">


<p id="status">
Starting...
</p>


<div class="bar">

<div
class="fill"
id="fill">
</div>

</div>


<div id="output"></div>


</div>


</div>


</main>


<script>


async function generate() {


    const prompt =
        document
        .getElementById("prompt")
        .value
        .trim();


    if (!prompt) {

        alert(
            "Please enter a prompt."
        );

        return;
    }


    document
        .getElementById("box")
        .style.display = "block";


    document
        .getElementById("output")
        .innerHTML = "";


    const response =
        await fetch(
            "/api/generate",
            {

                method: "POST",

                headers: {
                    "Content-Type":
                        "application/json"
                },

                body: JSON.stringify({

                    prompt,

                    ratio:
                        document
                        .getElementById("ratio")
                        .value,

                    quality:
                        document
                        .getElementById("quality")
                        .value,

                    duration:
                        Number(
                            document
                            .getElementById("duration")
                            .value
                        )

                })

            }
        );


    const data =
        await response.json();


    if (!response.ok) {

        alert(
            data.error ||
            "Generation failed"
        );

        return;
    }


    poll(data.jobId);

}


async function poll(id) {


    const response =
        await fetch(
            "/api/jobs/" + id
        );


    const job =
        await response.json();


    document
        .getElementById("fill")
        .style.width =
        job.progress + "%";


    document
        .getElementById("status")
        .textContent =
        job.status +
        " — " +
        job.progress +
        "%";


    if (
        job.status ===
        "completed"
    ) {


        if (job.videoUrl) {


            document
            .getElementById("output")
            .innerHTML = `

                <video
                    controls
                    src="${job.videoUrl}">
                </video>

            `;


        } else {


            document
            .getElementById("output")
            .innerHTML = `

                <p>
                Demo complete.
                Real AI video provider
                अभी connect करना बाकी है.
                </p>

            `;

        }


    } else {


        setTimeout(
            () => poll(id),
            1000
        );

    }

}

</script>


</body>

</html>PORT=3000

VIDEO_PROVIDER=demo

VIDEO_API_URL=

VIDEO_API_KEY=Prompt2Video/
│
├── package.json
├── server.js
├── .env
│
└── public/
    └── index.html
