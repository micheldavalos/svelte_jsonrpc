<script>
    import Speedometer from "svelte-speedometer"
    import { onMount } from "svelte";
    import Line from "svelte-chartjs/src/Line.svelte"

    let value = 0
    let token = 0
    let nombre = ''
    let nombre_02 = ''

    let x = 0;
    let y = 0;
    let chartValues = [];
    let chartLabels = [];
    let data = {
        labels: chartLabels,
        datasets: [{
            label: 'Revenue',
            backgroundColor: 'rgb(255, 99, 132)',
            borderColor: 'rgb(255, 99, 132)',
            data: chartValues
        }]
    };
    let options = {
        animation: false,
        //Boolean - If we want to override with a hard coded scale
        scaleOverride: true,
        //** Required if scaleOverride is true **
        //Number - The number of steps in a hard coded scale
        scaleSteps: 10,
        //Number - The value jump in the hard coded scale
        scaleStepWidth: 10,
        //Number - The scale starting value
        scaleStartValue: 0
    };

    setInterval(() => {

        if (token !== 0) {
            // console.log('aqui')
            postData('https://192.168.0.254/api/jsonrpc', [
                {
                    "jsonrpc": "2.0",
                    "method": "PlcProgram.Read",
                    "id": 1,
                    "params": {
                        "var": "\"Data_block_1\".nombre"
                    }
                }
            ])
                .then(data => {
                    // console.log(data); // JSON data parsed by `data.json()` call
                    nombre = data[0].result
                });
            postData('https://192.168.0.254/api/jsonrpc', [
                {
                    "jsonrpc": "2.0",
                    "method": "PlcProgram.Read",
                    "id": 1,
                    "params": {
                        "var": "\"Data_block_1\".SineWave"
                    }
                }
            ])
                .then(data1 => {
                    // console.log(data); // JSON data parsed by `data.json()` call
                    y = data1[0].result
                    x = x + 1
                    // y = y + 1
                    chartLabels.push(x.toString())
                    chartValues.push(y)
                    // console.log(chartValues)
                    data = {
                        labels: chartLabels,
                        datasets: [{
                            label: 'Revenue',
                            backgroundColor: 'rgb(255, 99, 132)',
                            borderColor: 'rgb(255, 99, 132)',
                            data: chartValues
                        }]
                    }
                });
        }
    }, 500)
    
    function click() {
        value++
    }
    function send() {
        if (token !== 0) {
            // console.log('aqui')
            postData('https://192.168.0.254/api/jsonrpc', [
                {
                    "jsonrpc": "2.0",
                    "method": "PlcProgram.Write",
                    "id": 1,
                    "params": {
                        "var": "\"Data_block_1\".nombre",
                        "value": nombre_02
                    }
                }
            ])
                .then(data => {
                    // console.log(data); // JSON data parsed by `data.json()` call
                    nombre = data[0].result
                });
        }
    }

    onMount(async () => {
        fetch('https://192.168.0.254/api/jsonrpc', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
                // 'Content-Type': 'application/x-www-form-urlencoded',
            },
            body: JSON.stringify([{
                id: 999,
                jsonrpc: "2.0",
                method: "Api.Login",
                params: {
                    user: "json",
                    password: "pegaduro2021"
                }
            }])
        })
            .then(response => response.json())
            .then(data => {
                console.log(data)
                token = data[0].result.token
                console.log(token)
        }).catch(error => {
            console.log(error)
        })
    })

    setInterval(() => {
        if (token !== 0) {
            // console.log('aqui')
            postData('https://192.168.0.254/api/jsonrpc', [
                {
                    "jsonrpc": "2.0",
                    "method": "PlcProgram.Read",
                    "id": 1,
                    "params": {
                        "var": "\"Data_block_1\".nombre"
                    }
                }
            ])
                .then(data => {
                   // console.log(data); // JSON data parsed by `data.json()` call
                    nombre = data[0].result
                });
            postData('https://192.168.0.254/api/jsonrpc', [
                {
                    "jsonrpc": "2.0",
                    "method": "PlcProgram.Read",
                    "id": 1,
                    "params": {
                        "var": "\"Data_block_1\".i"
                    }
                }
            ])
                .then(data => {
                    // console.log(data); // JSON data parsed by `data.json()` call
                    value = data[0].result
                });
        }
    }, 500)

    // Ejemplo implementando el metodo POST:
    async function postData(url = '', data = {}) {
        // Opciones por defecto estan marcadas con un *
        const response = await fetch(url, {
            method: 'POST', // *GET, POST, PUT, DELETE, etc.
            headers: {
                'X-Auth-Token': token,
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(data) // body data type must match "Content-Type" header
        });
        return response.json(); // parses JSON response into native JavaScript objects
    }
</script>

<Speedometer
    maxValue={500}
    value={value}
    segments={1}
    needleColor="black"
/>
<button on:click={click}>
    Incrementa
</button>

<p></p>
Variable nombre en PLC: {nombre}
<p></p>

<input bind:value={nombre_02}>
<button on:click={send}>Send</button>

<Line data={data} options={options}/>
<style>

</style>