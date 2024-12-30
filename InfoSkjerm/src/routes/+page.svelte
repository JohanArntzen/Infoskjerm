<script lang="ts">
    import { onMount } from 'svelte';

    const AVGANGER_API_URL = "https://api.entur.io/journey-planner/v3/graphql";
    const STOPP_STED_ID = "NSR:StopPlace:6435";

    let stoppNavn = '';
    let avganger: any[] = [];
    let avgangerLaster = true;
    let avgangerFeil = '';

    // GraphQL-spørring for å hente bussavganger
    const avgangerSpørring = `
    {
        stopPlace(id: "${STOPP_STED_ID}") {
            name
            estimatedCalls(timeRange: 3600, numberOfDepartures: 5) {
                aimedDepartureTime
                expectedDepartureTime
                destinationDisplay {
                    frontText
                }
                realtime
            }
        }
    }
    `;

    async function hentAvganger() {
        const headers = {
            "Content-Type": "application/json",
            "ET-Client-Name": "din-klient-navn" // Erstatt med ditt klientnavn
        };

        try {
            const response = await fetch(AVGANGER_API_URL, {
                method: "POST",
                headers: headers,
                body: JSON.stringify({ query: avgangerSpørring })
            });

            if (!response.ok) {
                throw new Error(`HTTP-feil! Status: ${response.status}`);
            }

            const data = await response.json();
            console.log("Hentet data:", data); // Logg de hentede dataene
            stoppNavn = data.data.stopPlace.name;
            avganger = data.data.stopPlace.estimatedCalls;
        } catch (err) {
            console.error("Feil ved henting av data:", err);
            avgangerFeil = "Kunne ikke laste avganger. Vennligst prøv igjen senere.";
        } finally {
            avgangerLaster = false;
        }
    }

    onMount(() => {
        hentAvganger();
    });

    const VÆR_API_URL = "https://api.met.no/weatherapi/locationforecast/2.0/compact";
    const BREDDEGRAD = 59.9; // Erstatt med ønsket breddegrad
    const LENGDEGRAD = 10.8; // Erstatt med ønsket lengdegrad

    let værData: any = null;
    let værLaster = true;
    let værFeil = "";

    // Hent værdata fra Yr API
    async function hentVær() {
        const headers = {
            "User-Agent": "din-app-navn (din-epost-eller-url)"
        };

        try {
            const response = await fetch(
                `${VÆR_API_URL}?lat=${BREDDEGRAD}&lon=${LENGDEGRAD}`,
                { headers }
            );

            if (!response.ok) {
                throw new Error(`HTTP-feil! Status: ${response.status}`);
            }

            const data = await response.json();
            console.log("Hentet værdata:", data);
            værData = data;
        } catch (err) {
            console.error("Feil ved henting av data:", err);
            værFeil = "Kunne ikke laste værdata. Vennligst prøv igjen senere.";
        } finally {
            værLaster = false;
        }
    }

    // Kjør hentVær når komponenten monteres
    onMount(() => {
        hentVær();
    });

</script>




<div class="background">
    <div class="info">
        <div class="busavganger">
            <h1>Neste Avganger</h1>
            {#if avgangerLaster}
                <div>Laster avganger...</div>
            {:else if avgangerFeil}
                <div>{avgangerFeil}</div>
            {:else}
                <div>
                    <h2>{stoppNavn}</h2>
                    <ul>
                        {#each avganger as avgang}
                            <li>
                                Buss til {avgang.destinationDisplay.frontText}: 
                                {new Date(avgang.expectedDepartureTime).toLocaleTimeString([], { hour: "2-digit", minute: "2-digit", hour12: false })} 
                                ({avgang.realtime ? "Sanntid" : "Planlagt"})
                            </li>
                        {/each}
                    </ul>
                </div>
            {/if}
        </div>
        <div class="værmelding">
            <h1>Værmelding</h1>
            {#if værLaster}
                <div>Laster værdata...</div>
            {:else if værFeil}
                <div>{værFeil}</div>
            {:else}
                <div>
                    <h2>Værmelding for Breddegrad {BREDDEGRAD}, Lengdegrad {LENGDEGRAD}</h2>
                    <ul>
                        {#each værData.properties.timeseries.slice(0, 5) as værmelding}
                            <li>
                                <strong>{new Date(værmelding.time).toLocaleString([], { hour: "2-digit", minute: "2-digit", hour12: false })}:</strong>
                                Temperatur: {værmelding.data.instant.details.air_temperature}°C,
                                Vind: {værmelding.data.instant.details.wind_speed} m/s
                            </li>
                        {/each}
                    </ul>
                </div>
            {/if}
        </div>
    </div>
    <div class="fill"></div>
</div>




<style>
    h2 {
        font-size: 3.5rem;
        margin: 1rem 0;
    }
    ul {
        list-style-type: none;
        padding: 0;
    }
    li {
        margin: 1rem 0;
    }

    .background {
        height: 3840px;
        width: 2160px;
        margin: 0;
        left: 0;
        top: 0;
        position: fixed;
        background-image: url('/background.svg');
        background-size: cover;
        background-position: center;
        background-repeat: no-repeat;
        display: flex;
        flex-direction: column;
        font-size: 3rem;
        font-family: futura-pt, sans-serif;
        font-weight: 700;
        color: white;
    }

    .info {
        height: 55%;
        width: 100%;
        display: flex;
        align-items: center;
        flex-direction: column;
        justify-content: space-around;
    }

    .fill {
        height: 45%;
        width: 100%;
    }

    .busavganger, .værmelding {
        background-color: rgba(0, 0, 0, 0.5);
        border-radius: 8rem;
        width: 70%;
        padding: 1rem 6rem;
    }
</style>