<script>
    import { fade } from 'svelte/transition';
    
    let fechaActual = new Date();
    let mes = fechaActual.getMonth();
    let anio = fechaActual.getFullYear();
    let eventos = {};
    let modoAgrupacion = "mes";

    function obtenerDiasDelMes(mes, anio) {
        const fechaInicio = new Date(anio, mes, 1); // Primer día del mes
        const fechaFin = new Date(anio, mes + 1, 0); // Último día del mes
        const dias = [];

        const diaDeInicio = (fechaInicio.getDay() + 6) % 7; // Reorganizar para que 0 sea lunes
        for (let i = 0; i < diaDeInicio; i++) {
            dias.push(null); 
        }

        for (let i = 1; i <= fechaFin.getDate(); i++) {
            dias.push(new Date(anio, mes, i));
        }
        while (dias.length < 42) {
            dias.push(null);
        }
        console.log(`Mes: ${mes + 1}, Días: ${dias.length}`, dias);
        return dias;
    }
    $: dias = obtenerDiasDelMes(mes, anio);
    function cambiarMes(direccion) {
        mes += direccion;
        if (mes < 0) {
            mes = 11;
            anio--;
        } else if (mes > 11) {
            mes = 0;
            anio++;
        }
    }
    function cargarEventos() {
        const datosGuardados = localStorage.getItem("eventos");
        if (datosGuardados) {
            eventos = JSON.parse(datosGuardados);
        }
    }
    cargarEventos();

    function agruparPorMes(eventos) {
    const agrupados = {};

    for (const fecha in eventos) {
        const mes = new Date(fecha).toLocaleDateString("es-ES", { month: "long", year: "numeric" });
        if (!agrupados[mes]) {
            agrupados[mes] = []; 
        }

        eventos[fecha].forEach((descripcion) => {
            agrupados[mes].push({ fecha, descripcion });
        });
    }

    return agrupados;
    }

    function obtenerEventosOrdenados() {
        const listaEventos = [];
        for (const clave in eventos) {
            eventos[clave].forEach((evento) => {
                listaEventos.push({ fecha:clave, descripcion: evento});
            });
        }
        listaEventos.sort((a, b) => new Date(a.fecha).getTime() - new Date(b.fecha).getTime());        return listaEventos;
    }
    function guardarEventos() {
        console.log("Guardando eventos en el localStorage:", eventos)
        localStorage.setItem("eventos", JSON.stringify(eventos));
    }

    function formatFechaLocal(fecha) {
        return fecha.toLocaleDateString("es-ES", {
            year: "numeric",
            month: "2-digit",
            day: "2-digit"
        }).split('/').reverse().join('-'); 
    }

    function agregarEvento(fecha) {
        const clave = formatFechaLocal(fecha);
        const nuevoEvento = prompt(`Añadir un evento para el día ${fecha.toLocaleDateString()}:`);

        if (nuevoEvento) {
            if (!eventos[clave]) {
                eventos[clave] = [];
            }
            eventos[clave].push(nuevoEvento);
            guardarEventos();
        }
    }
    function editarEvento(evento) {
        const nuevoTexto = prompt("Editar evento: ", evento.descripcion);
        if (nuevoTexto) {
            const clave = evento.fecha;
            const index = eventos[clave].indexOf(evento.descripcion);
            if (index !== 1) {
                eventos[clave][index] = nuevoTexto;
                guardarEventos();
            }
        }
    }
    function eliminarEvento(evento) {
        const clave = evento.fecha; // Fecha del evento
        const index = eventos[clave].indexOf(evento.descripcion); // Buscar el índice del evento en la lista

        if (index !== -1) {
            eventos[clave].splice(index, 1); // Eliminar el evento del array

            // Si no quedan eventos en esta fecha, elimina la clave
            if (eventos[clave].length === 0) {
                delete eventos[clave];
            }

            // Reasignar `eventos` para que Svelte detecte el cambio
            eventos = { ...eventos };
            guardarEventos(); // Guardar en localStorage
        }
        console.log("Eventos actualizados:", eventos)
    }

</script>
<div class="layout">
    <div class="calendario">
        <div class="navegacion">
            <button on:click={() => cambiarMes(-1)}>Anterior</button>
            <h2>{mes + 1} - {anio}</h2>
            <button on:click={() => cambiarMes(1)}>Siguiente</button>
        </div>
        <div class="dias">
            <div class="dia-semana">Lun</div>
            <div class="dia-semana">Mar</div>
            <div class="dia-semana">Mié</div>
            <div class="dia-semana">Jue</div>
            <div class="dia-semana">Vie</div>
            <div class="dia-semana">Sáb</div>
            <div class="dia-semana">Dom</div>
            {#each dias as dia, index}
                {#if dia}
                    <button class="dia {eventos[dia.toISOString().split('T')[0]] ? 'evento' : ''}" on:click={() => agregarEvento(dia)} type="button">
                        <div>{dia.getDate()}</div>
                            {#if eventos[formatFechaLocal(dia)]}
                                <div class="indicador-evento"></div>
                            {/if}
                    </button>
                {:else}
                    <div class="diaVacio"></div>
                {/if}
            {/each}
        </div>
</div>
    <div class="lista-eventos">
        <h3>Lista de eventos</h3>
        {#if modoAgrupacion === "mes" && Object.entries(agruparPorMes(eventos)).length > 0}
            {#each Object.entries(agruparPorMes(eventos)) as [mes, eventosMes]}
                <h4>{mes}</h4>
                <ul>
                    {#each eventosMes as evento (evento.descripcion)}
                    <li in:fade="{{ duration: 300 }}" out:fade="{{ duration: 300 }}">
                        <div>
                        <strong>{new Date(evento.fecha).toLocaleDateString()}:</strong> {evento.descripcion}
                        </div>
                        <div class="botones">
                        <button on:click={() => editarEvento(evento)}>Editar</button>
                        <button on:click={() => eliminarEvento(evento)}>Eliminar</button>
                        </div>
                    </li>
                    {/each}
                </ul>
            {/each}
        {:else}
            <p>No hay eventos programados.</p>
        {/if}
    </div>
</div>