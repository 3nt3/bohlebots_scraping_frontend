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

    type Entry = [number, string, number];

    let entries: Entry[] = await fetch("htts://bohlebots.3nt3.de/api").then(
      (res) => res.json()
    );

    let names = [...new Set(entries.map((x) => x[1]))];

    let colors = [
      "#00a8ff",
      "#9c88ff",
      "#fbc531",
      "#4cd137",
      "#487eb0",
      "#e84118",
      "#7f8fa6",
      "#273c75",
      "#353b48",
    ];

    let data = {
      datasets: names.map((name, i) => ({
        label: name,
        data: entries
          .filter((entry) => entry[1] === name)
          .map((entry) => ({
            x: entry[2] * 1000,
            y:
              new Date(entry[2] * 1000).getHours() +
              new Date(entry[2] * 1000).getMinutes() / 60,
          })),
        backgroundColor: colors[i],
        pointRadius: 5,
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
  });
</script>

<main>
  <canvas bind:this={chartCanvas} />
</main>

<style>
</style>
