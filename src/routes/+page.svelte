<script lang="ts">
	let lat = 52.1943,
		long = 0.1374,
		incrementMins = 10,
		gaugeRows = 35,
		gaugeSts = 27,
		localTime = true,
		rowsPerDay = 1,
		willBeTube = true;
	import { getSunrise, getSunset } from 'sunrise-sunset-js';

	interface SunTimesData {
		sunrise: Date;
		sunset: Date;
	}

	// TODO let user specify a different year?
	$: times = getSolarEventsForYear(lat, long, 2023);

	const endOfDayMinutes = 24 * 60;

	function getSolarEventsForYear(latitude: number, longitude: number, year: number) {
		const result = new Map();
		const start = new Date(year, 0, 1).getTime();
		for (let i = 0; i < 366; i++) {
			const d = new Date(start + i * 24 * 60 * 60 * 1000);
			if (d.getFullYear() > year) break; // For non-leap year
			result.set(d.toLocaleDateString('sv-SV'), {
				sunrise: getSunrise(latitude, longitude, d),
				sunset: getSunset(latitude, longitude, d)
			});
		}
		return result;
	}

	function dateTimeToStitchCount(dateTime: Date, increments: number, useLocalTime: boolean) {
		const time = useLocalTime
			? dateTime.getHours() * 60 + dateTime.getMinutes()
			: dateTime.getUTCHours() * 60 + dateTime.getUTCMinutes();
		return Math.round(time / increments);
	}

	function whichSide(date: Date) {
		const daysIntoYear =
			(Date.UTC(date.getFullYear(), date.getMonth(), date.getDate()) -
				Date.UTC(date.getFullYear(), 0, 0)) /
			24 /
			60 /
			60 /
			1000;
		//https://stackoverflow.com/a/40975730
		return daysIntoYear % 2 !== 0 ? 'RS' : 'WS';
	}

	function renderRow(day: SunTimesData, increments: number, useLocalTime: boolean, tube: boolean) {
		const sunriseUnits = dateTimeToStitchCount(day.sunrise, increments, useLocalTime);
		const sunsetUnits = dateTimeToStitchCount(day.sunset, increments, useLocalTime);

		let [morningSts, daySts, eveningSts] = [
			sunriseUnits,
			sunsetUnits - sunriseUnits,
			endOfDayMinutes / increments - sunsetUnits
		];
		if (tube) {
			// account for end stitches getting lost in the seam
			morningSts = morningSts + 1;
			eveningSts = eveningSts + 1;
		}
		const side = whichSide(day.sunrise);
		// Handle purl (WS) rows being worked backwards!
		return `(${side}) ${side == 'RS' ? 'knit' : 'purl'} ${
			side == 'RS' ? morningSts : eveningSts
		}sts in NC, ${daySts}sts in DC, ${side == 'RS' ? eveningSts : morningSts}sts in NC`;
	}
</script>

<h1 class="text-center">Daylight</h1>
<p class="italic text-center text-xl">
	Visualise the sun times at your location using fibre crafts.
</p>

<p class="bg-yellow-500/30 shadow-sm p-4 rounded-md">
	<svg
		xmlns="http://www.w3.org/2000/svg"
		fill="none"
		viewBox="0 0 24 24"
		stroke-width="1.5"
		stroke="currentColor"
		class="w-6 h-6"
	>
		<path
			stroke-linecap="round"
			stroke-linejoin="round"
			d="M12 9v3.75m-9.303 3.376c-.866 1.5.217 3.374 1.948 3.374h14.71c1.73 0 2.813-1.874 1.948-3.374L13.949 3.378c-.866-1.5-3.032-1.5-3.898 0L2.697 16.126zM12 15.75h.007v.008H12v-.008z"
		/>
	</svg>
	Work in progress! I'm building this in public. There's a good chance it's currently broken, and it
	definitely isn't feature-complete! You can see the code
	<a href="https://github.com/caro401/daylight">on GitHub</a>, and discussion of what it should do
	<a href="https://fosstodon.org/@carofyi/111175915171072111">in this thread on Mastodon</a>. It
	definitely doesn't work if you're in a place that doesn't have a sunrise and a sunset every day.
</p>

<h2>What is this?</h2>
<p>
	A pattern recipe for an intarsia knit cowl or scarf. Or you could be inventive and turn a knitted
	rectangle into whatever you like, or make the thing using crochet or weaving. I'll give full(ish)
	instructions for a knit cowl.
</p>
<h2>What you'll need</h2>
<ul>
	<li>
		2 colours of yarn, one (NC) to represent night, and one (DC) to represent day. They should be
		the same yarn weight. Estimates for how much you'll need will come later.
	</li>
	<li>Whatever size needles gave you the gauge you specified below.</li>
	<li>
		Some waste yarn to do a provisional cast on (and a crochet hook if that's your preferred method)
	</li>
	<li>A darning needle</li>
	<li>
		(Optional) Some beads, which you could use to mark months, seasons, equinoxes or whatever you
		fancy. <a
			href="https://www.susannawinter.net/post/2020/04/17/how-to-add-beads-to-knitting-tutorial"
			>Here's some instructions for various ways to add beads</a
		>, get creative!
	</li>
</ul>
<div class="border-4 p-4 border-yellow-500 rounded-md">
	<h2 class="-mt-1">Pattern Options</h2>
	<p class="text-sm italic">
		The example values are for the daylight at Cambridge Station (UK), on a standard(ish) gauge for
		knitting 4ply/fingering yarn.
	</p>
	<label>Latitude <input bind:value={lat} type="text" /></label>
	<label>Longitude <input bind:value={long} type="text" /></label> <br />
	<label class="block my-4"
		>Minutes represented per stitch <input
			bind:value={incrementMins}
			type="text"
			class="w-20"
		/></label
	>

	<!-- TODO handle pattern generation for >1 row/day!
		 <label class="block my-4"
		>Rows per day <input bind:value={rowsPerDay} type="text" class="w-20" /></label
	> -->
	<label
		>Show times in your current local time including daylight savings? <input
			type="checkbox"
			bind:checked={localTime}
		/> (otherwise, times in UTC)</label
	><br />
	<label
		>Will you make it into a tube rather than leaving it flat with the intarsia back exposed? <input
			type="checkbox"
			bind:checked={willBeTube}
		/></label
	><br />
	<p>
		Gauge for 10x10cm square in stocking stitch is <label
			><input type="text" class="w-20" bind:value={gaugeRows} /> rows</label
		>
		by
		<label><input type="text" class="w-20" bind:value={gaugeSts} /> stitches</label>.
	</p>
	<p>
		This gives you {endOfDayMinutes / incrementMins} stitches per row (~{Math.round(
			(endOfDayMinutes / incrementMins) * (10 / gaugeSts)
		)}cm wide), and 365 rows (~{Math.round(365 * rowsPerDay * (10 / gaugeRows))}cm long)
	</p>
	<p>
		Play with balancing your gauge, the number of minutes per stitch and rows per day to get
		dimensions you're happy with.
		{#if willBeTube}
			You've said you'll make it into a tube, so the final width will be <span class="font-bold"
				>half</span
			> your row width.
		{/if}
	</p>
</div>

<h2>Pattern</h2>
<p>
	⚠️ Write down the values you provided in the options above, these aren't saved (yet) and will be
	reset if you refresh the page. ⚠️
</p>
<p>
	Start with a provisional cast on of your choice. <a
		href="https://skeinreaction.com/2022/01/18/crochet-provisional-cast-on/"
		>Here's some instructions for how to do a provisional cast on</a
	>.
</p>
<p>
	Knit a big rectangle in stocking stitch (knit the right side, purl the wrong side). Use the <a
		href="https://www.thesprucecrafts.com/learn-to-knit-intarsia-2116388">intarsia technique</a
	> to change the yarn from night to day colour and back, according to the big set of colour instructions
	below.
</p>
<p>
	Stop just before the last row if you want to make a cowl, leave these stitches live on your
	needle.
</p>
<p>
	If your pattern options say so, seam the thing into a tube along the long edge, joining the middle
	of the nights together. This will hide your intarsia back and make the whole thing half as wide.
	Mattress stitch probably works best, be careful not to pull too tight so it doesn't gather at the
	seam.
</p>

<p>
	If you're making a cowl, undo your provisional cast on, and graft the stitches you left live it to
	the first row, using the colours as defined for the last row. <a
		href="https://skeinreaction.com/2022/01/18/grafting-or-kitchener-stitch/"
		>Here's some instructions for how to graft.</a
	> If you're making a scarf, graft closed each end of the tube individually.
</p>

<h3>Colours</h3>
<p>
	Start with a right-side (RS) row, knit all stitches using the colours as described. Next row (WS)
	turn, purl all the stitches
</p>
{#each times as [date, data]}
	<p class="whitespace-nowrap text-xs max-w-none font-mono">
		{new Date(date).toLocaleDateString('en-GB', { month: 'short', day: '2-digit' })}:
		{renderRow(data, incrementMins, localTime, willBeTube)}
	</p>
{/each}
