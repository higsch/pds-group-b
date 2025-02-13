<script>
  import { scaleLinear } from 'd3';
  import {arc} from 'd3';

  let arcgenerator = arc()

  let minuteHand;
  let clockFace;

  let isDragging = false;

  let lockedTime = $state();
  let clockColor = $state('black');

  let correctTime = 60;

  let showResult = $state(false);

  let circlesegment = $state();

  const colorScale = scaleLinear()
    .domain([0, 60])
    .range(['green', 'red']);

  function angleToMinutes(radians) {
    // Normalize the angle to be within 0 and 2π
    const normalizedRadians = ((radians % (2 * Math.PI)) + (2 * Math.PI)) % (2 * Math.PI);

    // Convert radians to degrees (360 degrees corresponds to 2π radians)
    const degrees = (normalizedRadians * 360) / (2 * Math.PI);

    // Convert degrees to minutes (360 degrees corresponds to 60 minutes)
    const minutes = (degrees / 360) * 60;

    return minutes;
  }


  function handleClockContainerMouseDown(event) {
    isDragging = true;
  }

  function handleClockContainerMouseMove(event) {
    if (isDragging) {
      //const rect = clockContainer.getBoundingClientRect();
      //const centerX = rect.left + 100; // Mittelpunkt der Uhr (X)
      //const centerY = rect.top + 100; // Mittelpunkt der Uhr (Y)

      const centerX = 100;
      const centerY = 100;

      const dx = event.offsetX - centerX;
      const dy = event.offsetY - centerY;

      const angle = Math.atan2(dy, dx); // Winkel in Radiant

function normalizeAngle(radians) {
  return ((radians % (2 * Math.PI)) + (2 * Math.PI)) % (2 * Math.PI);
}

circlesegment = arcgenerator({
  innerRadius: 0,
  outerRadius: 92,
  startAngle: normalizeAngle(0),
  endAngle: normalizeAngle(angle + Math.PI/2),
});

      // Neue Position des Minutenzeigers berechnen
      const x2 = 100 + 70 * Math.cos(angle); // Endpunkt (X)
      const y2 = 100 + 70 * Math.sin(angle); // Endpunkt (Y)

      // Minutenzeiger aktualisieren
      minuteHand.setAttribute('x2', x2);
      minuteHand.setAttribute('y2', y2);
    }
  }

  function handleClockContainerMouseUp() {
    isDragging = false;
  }

  function handleButtonClick() {
    if (isDragging) return; // Verhindern, dass während des Draggens der Button ausgelöst wird

    const x2 = minuteHand.getAttribute('x2');
    const y2 = minuteHand.getAttribute('y2');

    // Winkel des aktuellen Zeigers berechnen
    const dx = x2 - 100;
    const dy = y2 - 100;
    const lockedAngle = Math.atan2(dy, dx) + Math.PI / 2;
    lockedTime = angleToMinutes(lockedAngle);

    // Vergleichswert (12:45 = 270° Minutenzeiger)
    const correctTime = 60;
    const correctAngle = (correctTime / 60) * 2 * Math.PI; // 45 Minuten in Radiant

    // Richtige Uhrzeit anzeigen
    const correctHand = document.createElementNS(
      'http://www.w3.org/2000/svg',
      'line'
    );
    correctHand.setAttribute('x1', '100');
    correctHand.setAttribute('y1', '100');
    correctHand.setAttribute(
      'x2',
      100 + 70 * Math.cos(correctAngle - Math.PI / 2)
    );
    correctHand.setAttribute(
      'y2',
      100 + 70 * Math.sin(correctAngle - Math.PI / 2)
    );
    correctHand.setAttribute('stroke', 'red');
    correctHand.setAttribute('stroke-width', '2');
    document.querySelector('svg').appendChild(correctHand);

    // Change clock color depending on result
    clockColor = colorScale(Math.abs(lockedTime - correctTime));

    showResult = true;
  }
</script>

<main>
  <svg
    width="200"
    height="200"
    viewBox="0 0 200 200"
    onmousedown={handleClockContainerMouseDown}
    onmousemove={handleClockContainerMouseMove}
    onmouseup={handleClockContainerMouseUp}
  >
    <!-- Äußerer Kreis -->
    <circle bind:this={clockFace} id="clockFace" cx="100" cy="100" r="95" stroke={clockColor} fill="none" stroke-width="5" />
    <line x1="100" y1="10" x2="100" y2="20" stroke="black" stroke-width="2" />
    <line x1="100" y1="180" x2="100" y2="190" stroke="black" stroke-width="2" />
    <line x1="10" y1="100" x2="20" y2="100" stroke="black" stroke-width="2" />
    <line x1="180" y1="100" x2="190" y2="100" stroke="black" stroke-width="2" />

    <!-- Weitere Stundenmarkierungen -->
    <line x1="140" y1="27" x2="135" y2="38" stroke="black" stroke-width="2" />
    <line x1="173" y1="60" x2="165" y2="70" stroke="black" stroke-width="2" />
    <line x1="173" y1="140" x2="165" y2="130" stroke="black" stroke-width="2" />
    <line x1="140" y1="173" x2="135" y2="162" stroke="black" stroke-width="2" />
    <line x1="60" y1="173" x2="65" y2="162" stroke="black" stroke-width="2" />
    <line x1="27" y1="140" x2="35" y2="130" stroke="black" stroke-width="2" />
    <line x1="27" y1="60" x2="35" y2="70" stroke="black" stroke-width="2" />
    <line x1="60" y1="27" x2="65" y2="38" stroke="black" stroke-width="2" />
  
  <path d = {circlesegment}
  fill= 'grey',
  opacity= 0.5
  transform= 'translate(100 100)'></path>
    <!-- Minutenzeiger -->
    <line bind:this={minuteHand} id="minuteHand" x1="100" y1="100" x2="100" y2="30" stroke="black" stroke-width="7" stroke-linecap="round" />

    <!-- Mittelpunkt -->
    <circle cx="100" cy="100" r="3" fill="black" />
  </svg>

  <button id="lockButton" onclick={handleButtonClick}>Zeit einloggen</button>
  
  {#if showResult}
    <div id="result" style="margin-top: 10px; font-size: 16px;">
      <span>Du hast {Math.round(lockedTime)} Minuten geschätzt.</span>
      <span>Richtig sind {correctTime} Minuten.</span>
    </div>
  {/if}
</main>

<style>
@import url('https://fonts.googleapis.com/css2?family=Outfit:wght@400;700&display=swap');

  #result {
    display: flex;
    flex-direction: column;
    color: white;
  }
  main {
    display: flex;
    align-items: center;
    flex-direction: column;
    justify-content: center;
    font-family: Outfit;
    gap: 5px;
   

  }
</style>
