<script>
	import addDays from 'date-fns/addDays';
	import format from 'date-fns/format';
    import formatNumber from 'format-number';

    const formatter = formatNumber();

    let hazards = 0, countOfSpaceObjects = 0;
    let spaceObjects = [];

	function getDate(d = new Date()) {
	    return d.toJSON().split('T')[0];
    }
    const day = getDate(addDays(new Date(), 1));
	const formatDate = format(addDays(new Date(), 1), 'EEEE d-MMM');

    let promise = fetch(`https://api.nasa.gov/neo/rest/v1/feed?start_date=${getDate()}&api_key=DEMO_KEY`, {
        method: 'GET',
        mode: 'cors'
    })
        .then((response) => response.json())
        .then(data => {
           getData(data.near_earth_objects[day]);
           spaceObjects = data.near_earth_objects[day].sort((a) => (a.is_potentially_hazardous_asteroid ? -1 : 1));
           if (hazards > 0) {
               document.title = `${hazards} potential HAZARDS 😱`;
           }    else {
               document.title = `No potential HAZARDS 👍`;
           }
        })

    function getData(data) {

        countOfSpaceObjects = data.length;
        for (let i = 0; i < data.length; i++) {
            if (data[i].is_potentially_hazardous_asteroid) {
                hazards++;
            }
        }
    }

    function titleForWaiting() {
        document.title = 'Counting potential earth HAZARDS…';
    }
</script>

<div class="main-block">
    {#await promise}
        <p use:titleForWaiting>
            Getting data from NASA right now to check whether something from space
            is going to hit us. One moment…
        </p>
    {:then data}
        <div>
            <p>
                {formatDate} there will be <strong>{countOfSpaceObjects}</strong> near misses
            </p>
            <hr />
            {#each spaceObjects as spaceObj}
                <div class={spaceObj.is_potentially_hazardous_asteroid ? 'is-hazard' : 'no-hazard'}>
                    <h2>{spaceObj.name.replace(/[()]/g, '')}</h2>
                <p>
                    Potentially hazardous?
                    {#if spaceObj.is_potentially_hazardous_asteroid}
                        <span class="hazard">
                            YES 😱
                        </span>
                    {:else}
                        <span class="hazard">
                            nope
                        </span>
                    {/if}
                </p>
                <p>
                    Misses {spaceObj.close_approach_data[0].orbiting_body} tomorrow at
                    {format(spaceObj.close_approach_data[0].epoch_date_close_approach, 'h:mmaaaa')}
                    by {formatter(parseInt(spaceObj.close_approach_data[0].miss_distance.miles, 10))} miles whilst travelling
                    at {formatNumber({truncate: 0})(spaceObj.close_approach_data[0].relative_velocity.miles_per_hour)} mph
                </p>
                <p class="more">
                    <a href={spaceObj.nasa_jpl_url} target="_blank">
                        Find out more
                    </a>
                </p>
                </div>
            {/each}
        </div>
    {/await}
</div>
