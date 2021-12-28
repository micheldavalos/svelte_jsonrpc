<script>
    import Speedometer from "svelte-speedometer"
    import { onMount } from "svelte";
    let value = 0
    let token = 0
    let nombre = ''
    
    function click() {
        value++
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
        }
    }, 1000)

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
/>
<button on:click={click}>
    Incrementa
</button>

{nombre}
<style>

</style>