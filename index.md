---
layout: default
---

<input type="checkbox" id="reverse"> reverse <br />
<input type="file" id="files" multiple> <br />
<button id="submit">Process</button>

<div id="chart" style="height:80%"></div>

<script type="text/javascript">
    google.charts.load('current', {'packages':['annotatedtimeline']});

    function drawChart(results) {
        var data = new google.visualization.DataTable();
        data.addColumn('date', 'Date');
        data.addColumn('number', 'Speed');
        data.addColumn('number', 'Power (100W)');
        data.addColumn('number', 'Voltage');
        data.addColumn('number', 'Current');
        data.addColumn('number', 'Temperature');
        data.addColumn('number', 'Trip');
        data.addColumn('number', 'Odometer');
        data.addRows(results);

        var chart = new google.visualization.AnnotatedTimeLine(document.getElementById('chart'));
        chart.draw(data, {displayAnnotations: true});
    }
</script>
<script type="text/javascript">
    var config = {
        delimiter: "",
        header: false,
        dynamicTyping: true,
        skipEmptyLines: true,
        preview: 0,
        step: undefined,
        encoding: "",
        worker: true,
        comments: "",
        complete: completeFn,
        error: errorFn,
        download: false,
    };

    function completeFn(results) {
        console.log(results);
        var reverse = $('#reverse').is(':checked') ? -1 : 1;

        google.charts.setOnLoadCallback(function () {
            drawChart(results.data.map(function (data) {
                data[0] = new Date(data[0]);
                data[1] *= reverse;
                data[2] *= reverse / 100;
                data[3]  = data[3];
                data[4] *= reverse;
                data[5]  = data[5];
                delete data[6];
                delete data[7];
                return data;
            }))
        });
    }

    function errorFn (error, file) { console.log(error, file); }

    $('#submit').click(function (e) {
        if (! $('#files')[0].files.length) return;
        $('#files').parse({
            config: config,
            before: function(file, inputElem)
            {
                console.log("Parsing file...", file);
            },
            error: function(err, file)
            {
                console.log("ERROR:", err, file);
            },
            complete: function()
            {
                console.log("Done with all files");
            }
        });
    });

</script>