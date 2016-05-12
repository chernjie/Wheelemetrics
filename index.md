---
layout: default
---

<label><input type="checkbox" id="reverse" checked="checked"> reverse</label>

<div id="chart" style="height:80%"></div>

<script type="text/javascript">
    google.charts.load('current', {'packages':['annotatedtimeline']});

    var config = {
        header: false,
        dynamicTyping: true,
        skipEmptyLines: true,
        worker: true,
        complete: completeFn,
        error: function errorFn (error, file) {
            console.error("errorFn", error, file);
        },
    };

    function completeFn(results) {
        console.log(results);

        var reverse = $('#reverse').is(':checked') ? -1 : 1;
        var rows = results.data.map(processInput);
        var columns = [
            { type: 'date',   name: 'Date' },
            { type: 'number', name: 'Speed'},
            { type: 'number', name: 'Power (100W)' },
            { type: 'number', name: 'Voltage' },
            { type: 'number', name: 'Current' },
            { type: 'number', name: 'Temperature' },
            { type: 'number', name: 'Trip' },
            { type: 'number', name: 'Odometer' },
        ];
        var element = document.getElementById('chart');

        function processInput(data) {
            data[0] = new Date(data[0]);
            data[1] *= reverse;
            data[2] *= reverse / 100;
            data[3]  = data[3];
            data[4] *= reverse;
            data[5]  = data[5];
            delete data[6];
            delete data[7];
            return data;
        }

        google.charts.setOnLoadCallback(function () {
            var data = new google.visualization.DataTable();
            columns.forEach(function (column) {
                data.addColumn(column.type, column.name);
            })
            data.addRows(rows);
            var chart = new google.visualization.AnnotatedTimeLine(element);
            chart.draw(data, {displayAnnotations: true});
        });
    }

    $('.wrapper').on('drag dragstart dragend dragover dragenter dragleave drop', function(event) {
        event.preventDefault();
        event.stopPropagation();
    }).on('drop', function (event) {
        event.preventDefault();
        var firstFile = event.originalEvent.dataTransfer.files[0];
        console.log('Dropped file', firstFile);
        config.file = firstFile;
        Papa.parse(firstFile, config);
    });

    var $reverse = $('#reverse');
    $reverse.parent().click(function (event) {
        $reverse.prop('checked', ! $reverse.prop('checked'));
        config.file && Papa.parse(config.file, config);
    });
</script>