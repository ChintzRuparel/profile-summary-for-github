<script>
    const UNKNOWN_LANGUAGE = "Unknown";

    function donutChart(objectName, data) {
        let canvas = document.getElementById(objectName);
        if (canvas === null) {
            return;
        }
        let userId = data.user.login;
        let labels = Object.keys(data[objectName]);

        /**
         * The language index of <code>UNKNOWN_LANGUAGE</code>
         * @type {Number}
         */
        let indexOfUnknownLanguage = -1;
        let values = Object.values(data[objectName]);
        let colors = createColorArray(labels.length);
        let tooltipInfo = null;
        window.languageColors = window.languageColors || {};
        if ("langRepoCount" === objectName) {
            // when the first language-set is loaded, set a color-profile for all languages
            labels.forEach((language, i) => languageColors[language] = colors[i]);
        }
        if (["langRepoCount", "langStarCount", "langCommitCount"].indexOf(objectName) > -1) {
            // if the dataset is language-related, load color-profile
            labels.forEach((language, i) => colors[i] = languageColors[language]);
            indexOfUnknownLanguage = labels.indexOf(UNKNOWN_LANGUAGE);
        }
        if (objectName === "repoCommitCount") {
            tooltipInfo = data[objectName + "Descriptions"]; // high quality programming
            arrayRotate(colors, 4); // change starting color
        }
        if (objectName === "repoStarCount") {
            tooltipInfo = data[objectName + "Descriptions"]; // high quality programming
            arrayRotate(colors, 2); // change starting color
        }
        new Chart(canvas.getContext("2d"), {
            type: "doughnut",
            data: {
                labels: labels,
                datasets: [{
                    data: values,
                    backgroundColor: colors
                }]
            },
            options: {
                animation: false,
                rotation: (-0.40 * Math.PI),
                legend: { // todo: fix duplication ?
                    position: window.innerWidth < 600 ? "bottom" : "left",
                    labels: {
                        fontSize: window.innerWidth < 600 ? 10 : 12,
                        padding: window.innerWidth < 600 ? 8 : 10,
                        boxWidth: window.innerWidth < 600 ? 10 : 12
                    }
                },
                tooltips: {
                    callbacks: {
                        afterLabel: function (tooltipItem, data) {
                            if (tooltipInfo !== null) {
                                return wordWrap(tooltipInfo[data["labels"][tooltipItem["index"]]], 45);
                            }

                            if (tooltipItem.index === indexOfUnknownLanguage) {
                                return "No specific language";
                            }
                        }
                    },
                },
                onClick: function (e, data) {
                    try {
                        let label = labels[data[0]._index];
                        const isUnknownLanguage = (data[0]._index === indexOfUnknownLanguage);
                        let canvas = data[0]._chart.canvas.id;
                        if (canvas === "repoStarCount" || canvas === "repoCommitCount") {
                            window.open("https://github.com/" + userId + "/" + label, "_blank");
                            window.focus();
                        } else {
                            if (!isUnknownLanguage) {
                                window.open("https://github.com/" + userId + "?utf8=%E2%9C%93&tab=repositories&q=&type=source&language=" + encodeURIComponent(label), "_blank");
                                window.focus();
                            }
                        }
                    } catch (ignored) {
                    }
                },
                onResize: function (instance) { // todo: fix duplication ?
                    instance.chart.options.legend.position = window.innerWidth < 600 ? "bottom" : "left";
                    instance.chart.options.legend.labels.fontSize = window.innerWidth < 600 ? 10 : 12;
                    instance.chart.options.legend.labels.padding = window.innerWidth < 600 ? 8 : 10;
                    instance.chart.options.legend.labels.boxWidth = window.innerWidth < 600 ? 10 : 12;
                }
            }
        });

        function createColorArray(length) {
            const colors = ["#1e81b0", "#eeeee4", "#e28743", "#eab676", "#76b5c5", "#21130d", "#873e23", "#d5817c"];
            let array = [...Array(length).keys()].map(i => colors[i % colors.length]);
            // avoid first and last colors being the same
            if (length % colors.length === 1) {
                array[length - 1] = colors[1];
            }
            return array;
        }

        function arrayRotate(arr, n) {
            for (let i = 0; i < n; i++) {
                arr.push(arr.shift());
            }
            return arr
        }

        function wordWrap(str, n) {
            if (str === null) {
                return null;
            }
            let currentLine = [];
            let resultLines = [];
            str.split(" ").forEach(word => {
                currentLine.push(word);
                if (currentLine.join(" ").length > n) {
                    resultLines.push(currentLine.join(" "));
                    currentLine = [];
                }
            });
            if (currentLine.length > 0) {
                resultLines.push(currentLine.join(" "));
            }
            return resultLines
        }
    }

    function lineChart(objectName, data) {
        new Chart(document.getElementById(objectName).getContext("2d"), {
            type: "line",
            data: {
                labels: Object.keys(data[objectName]),
                datasets: [{
                    label: "Commits",
                    data: Object.values(data[objectName]),
                    backgroundColor: "rgba(67, 142, 233, 0.2)",
                    borderColor: "rgba(67, 142, 233, 1)",
                    lineTension: 0
                }]
            },
            options: {
                maintainAspectRatio: false,
                animation: false,
                scales: {
                    xAxes: [{
                        display: false
                    }],
                    yAxes: [{
                        position: "right",
                        beginAtZero: true
                    }]
                },
                legend: {
                    display: false
                },
                tooltips: {
                    intersect: false
                }
            }
        });
    }
</script>
