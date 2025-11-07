---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---

<p align="center">
<img src="/assets/30r.jpg" alt="30R">
</p>
<p align="right" style="font-weight:regular;font-size:15px">
photo: by <a href="https://www.mattbeyer.com/">Matt Beyer</a>
</p>
<p align="center">
    <a style="font-weight:regular;font-size: 30px" href="https://discord.gg/nazp8Dnrva">Join the conversation on Discord!</a>
    <br>
    <br>
    Some of our tools...
    <br>
    <a style="font-weight:regular;font-size: 30px" href="https://coloradoaviators.net/AV-Scribe/">AV Scribe</a>
</p>

<div id="fuelhawk" align="center">
    <p id="fuelhawkTitle">
        <a style="font-weight:regular;font-size: 30px">Simple Fuelhawk conversion tool</a>
        <br>
        <a style="font-weight:bold;color: red">Based on interpolation between discrete readings (USE WITH CAUTION!)</a>
    </p>
    <div>
        <label for="fromModel">Fuelhawk model:</label>
        <select id="fromModel" onchange="setBounds();recalculate()"></select>
    </div>
    <div>
        <label for="toModel">Airplane model:</label>
        <select id="toModel" onchange="recalculate()"></select>
    </div>
    <div>
        <label for="reading">Fuelhawk reading:</label>
        <input type="number" id="reading" name="reading" value="0" min="0" max="100" step=".5" onchange="recalculate()">
    </div>
    <div>
        <a>Airplane reading:</a>
        <a id="convertedReading"></a>
    </div>
</div>

<script>
    class Model{
        constructor(readings, universals) {
            if (readings.length != universals.length) {
                throw "Readings and Universals must match in length";
            }
            this.readings = readings.toSorted((a, b) => a - b);
            this.universals = universals.toSorted((a, b) => a - b);
        }
        interpolate(reading, from, to) {
            var result = null;
            for (let i=1; i < from.length; i++){
                let above = from[i];
                if (reading > above) {
                    continue;
                }
                let below = from[i-1];
                let fraction = (reading - below) / (above - below);
                result = to[i - 1] + (to[i] - to[i - 1]) * fraction;
                return result;
            }
            return result;
        }
        toUniversal(reading) {
            return this.interpolate(reading, this.readings, this.universals);
        }
        fromUniversal(reading) {
            return this.interpolate(reading, this.universals, this.readings);
        }
    }
    const models = {
        "Universal": new Model(
            [0, 100],
            [0, 100],
        ),
        "Cessna 152": new Model(
            [0, .5, 2.5, 6, 10, 12],
            [0, 2, 3.5, 6, 9, 10.5],
        ),
        "Cessna 172 (26.5 gal)": new Model(
            [0, 13, 23, 26.5],
            [0, 6, 10.5, 12.25],
        ),
        "Cessna 172 (19 gal)": new Model(
            [0, 8.1, 16, 19],
            [.9, 5, 9, 10.5],
        ),
        "Cessna 182 (43.5 gal)": new Model(
            [0, 6, 9, 15, 25, 31, 43.5],
            [.5, 2.4, 3.5, 5.1, 7.5, 9, 13],
        ),
        "Cessna 182 (39 gal)": new Model(
            [1, 4, 7, 14.5, 16.5, 21, 26.5, 33, 36.5, 39],
            [0, .5, .9, 2, 2.4, 3.5, 5.1, 7.5, 9, 10.4],
        ),
        "Piper Archer": new Model(
            [0, 5, 10, 15, 20, 24],
            [0, 1.5, 4.5, 7.25, 10.2, 13.5],
        ),
    };
    function setBounds() {
        let fromModelName = document.getElementById("fromModel").value;
        let fromModel = models[fromModelName];
        let readingField = document.getElementById("reading");
        readingField.max = fromModel.readings[fromModel.readings.length - 1];
        readingField.min = fromModel.readings[0];
    }
    function recalculate() {
        let fromModelName = document.getElementById("fromModel").value;
        let toModelName = document.getElementById("toModel").value;
        let reading = document.getElementById("reading").value;
        let fromModel = models[fromModelName];
        let toModel = models[toModelName];
        let universalReading = fromModel.toUniversal(reading);
        let result = toModel.fromUniversal(universalReading);
        document.getElementById('convertedReading').innerHTML = result.toFixed(1);
    }
    window.onload = function setup() {
        var fromModelSelect = document.getElementById("fromModel");
        var toModelSelect = document.getElementById("toModel");
        for (const [key, value] of Object.entries(models)) {
            fromModelSelect.add(new Option(key, key));
            toModelSelect.add(new Option(key, key));
        }
        recalculate();
    }
</script>
