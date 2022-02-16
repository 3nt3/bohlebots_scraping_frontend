<script lang="ts">
  import {
    Chart,
    Legend,
    LinearScale,
    LineElement,
    PointElement,
    ScatterController,
    TimeScale,
    Tooltip,
  } from "chart.js";
  import "chartjs-adapter-date-fns";
  import { de } from "date-fns/locale";

  import { onMount } from "svelte";

  let chartCanvas: HTMLCanvasElement;
  let context;

  type Entry = [number, string, number];
  let entries: Entry[];

  const updateData = async () => {
    entries = await fetch("https://bohlebots.3nt3.de/api").then((res) =>
      res.json()
    );
  };

  function getRandomColor() {
    var letters = "0123456789ABCDEF";
    var color = "#";
    for (var i = 0; i < 6; i++) {
      color += letters[Math.floor(Math.random() * 16)];
    }
    return color;
  }

  onMount(async () => {
    context = chartCanvas.getContext("2d");

    Chart.register(
      ScatterController,
      LinearScale,
      PointElement,
      LineElement,
      TimeScale,
      Tooltip,
      Legend
    );

    await updateData();

    let names = [...new Set(entries.map((x) => x[1]))];

    let data = {
      datasets: names.map((name, i) => ({
        label: `${name} (${entries.filter((e) => e[1] === name).length})`,
        data: entries
          .filter((entry) => entry[1] === name)
          .map((entry) => ({
            x: entry[2] * 1000,
            y:
              new Date(entry[2] * 1000).getHours() +
              new Date(entry[2] * 1000).getMinutes() / 60,
          })),
        backgroundColor: getRandomColor(),
        pointRadius: 8,
      })),
    };
    // datasets: [
    //   {
    //     label: "scatter dataset",
    //     data: entries.map((entry) => {
    //       return {
    //         x: entry[2],
    //         y: 1,
    //       };
    //     }),
    //   },
    // ],
    let chart = new Chart(context, {
      type: "scatter",
      data: data,
      options: {
        locale: "de",
        plugins: {
          // tooltip: {
          //   callbacks: {
          //     label: (tooltipItem) =>
          //       `${tooltipItem.dataset.data[tooltipItem.dataIndex].y}`,
          //   },
          // },
          legend: {
            display: true,
            position: "bottom",
          },
        },
        scales: {
          yAxis: {
            type: "linear",
            position: "left",
            min: 0,
            max: 24,
            reverse: true,
          },
          xAxis: {
            type: "time",
            adapters: {
              date: {
                locale: de,
              },
            },
            max: new Date().getTime(),
            time: { unit: "day", tooltipFormat: "yyyy-MM-dd" },
          },
        },
      },
    });

    setInterval(() => updateData(), 10000);
  });
</script>

<main>
  <canvas bind:this={chartCanvas} />
</main>

<style>
</style>
