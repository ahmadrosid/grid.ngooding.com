<script lang="ts">
  import { page } from "$app/stores";
  import { onMount } from "svelte";

  let activity = [];
  let grid: (Date | null)[][] = Array(7).fill(Array(52).fill(null));
  let startDate: null | Date = null;

  function getMonthName(month: number) {
    if (month > 11) {
      // if month is greather than 11, I wan to reset it to january and return the month name
      month = month % 12;
    }
    const monthNames = [
      "Jan",
      "Feb",
      "Mar",
      "Apr",
      "May",
      "Jun",
      "Jul",
      "Aug",
      "Sep",
      "Oct",
      "Nov",
      "Dec",
    ];
    return monthNames[month];
  }

  function getDayName(day: number) {
    const dayNames = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];
    return dayNames[day];
  }

  $: slug = $page.params.slug;

  onMount(() => {
    if (!slug) {
      return;
    }
    fetch("https://opensheet.elk.sh/" + slug)
      .then((response) => response.json())
      .then((data) => {
        if (data.error) {
          throw new Error(data.error);
        }
        const dataDates = data.map((item: any) => {
          const date = new Date(item.Date as string);
          return formatDate(date);
        });

        function formatDate(date: Date) {
          return (
            date.getFullYear() +
            "-" +
            (date.getMonth() + 1) +
            "-" +
            date.getDate()
          );
        }

        function generateDateGrid() {
          const startDate = new Date(dataDates[0]);
          let grid = [];

          for (let i = 0; i < 7; i++) {
            let days = [];
            let currentDate = new Date(startDate);
            currentDate.setDate(currentDate.getDate() + i);

            for (let j = 0; j < 52; j++) {
              if (dataDates.includes(formatDate(currentDate))) {
                days.push(new Date(currentDate));
              } else {
                days.push(null);
              }
              currentDate.setDate(currentDate.getDate() + 7); // Move to next week
            }
            grid.push(days);
          }

          return grid;
        }

        grid = generateDateGrid();
        startDate = grid[0][0];
        activity = data;
      })
      .catch((error) => console.error("Error:", error));
  });
</script>

<div class="bg-slate-900 h-screen w-screen p-8">
  <div class="mx-auto max-w-[45rem] w-full">
    <div class="text-center pb-6">
      <h1 class="text-2xl font-bold text-white">LinkedIn Activity Grid</h1>
    </div>
    <div class="bg-white/5 border border-white/10 p-6 rounded-lg w-full">
      <table class="table-auto">
        {#if startDate}
          <thead>
            <tr class="pb-2">
              <td></td>
              {#each Array(12) as week, idx}
                <!-- set col span to 5 when the index is last index -->
                {#if idx < 8 && idx % 2 === 0}
                  <td class="text-xs text-white h-[20px]" colspan="5"
                    >{getMonthName(startDate.getMonth() + idx)}</td
                  >
                {:else}
                  <td class="text-xs text-white h-[20px]" colspan="4"
                    >{getMonthName(startDate.getMonth() + idx)}</td
                  >
                {/if}
              {/each}
            </tr>
          </thead>
          <tbody>
            {#each grid as days, idx}
              <tr>
                <td>
                  {#if idx === 0 || idx === 2 || idx === 4} 
                    <div class="pr-2 h-[10px]">
                        <span class="text-xs text-white relative">{getDayName(idx)}</span>
                    </div>
                  {/if}
                </td>
                {#each days as week}
                  <td>
                    <div
                      class="w-2.5 h-2.5 rounded-sm"
                      class:bg-green-500={week !== null}
                      class:bg-slate-700={week === null}
                    ></div>
                  </td>
                {/each}
              </tr>
            {/each}
          </tbody>
        {/if}
      </table>
    </div>
  </div>
</div>
