<script lang="ts">
  import { onMount } from "svelte";

  let activity = [];
  let grid: (Date | null)[][] = Array(7).fill(Array(52).fill(null));

  onMount(() => {
    fetch('https://opensheet.elk.sh/1zYJ-Ad-wMJeucV6jpXlQTz_I4Cjy1PgfGWUesXP_XJY/1')
    .then(response => response.json())
    .then(data => {
        const dataDates = data.map((item) => {
            const date = new Date(item.Date as string);
            return formatDate(date);
        });

        function formatDate(date: Date) {
            return (
            date.getFullYear() + "-" + (date.getMonth() + 1) + "-" + date.getDate()
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
        activity = data;
    })
    .catch(error => console.error('Error:', error));
  })
</script>

<div class="bg-slate-900 h-screen w-screen p-8">
  <div class="mx-auto max-w-2xl">
    <div class="text-center pb-6">
      <h1 class="text-2xl font-bold text-white">LinkedIn Activity Grid</h1>
    </div>
    <div
      class="bg-white/5 border border-white/10 p-6 rounded-lg overflow-x-auto"
    >
      <table>
        <tbody>
          {#each grid as days}
            <tr>
              {#each days as week}
                <td
                  ><div
                    class="w-2.5 h-2.5 rounded-sm"
                    class:bg-green-500={week !== null}
                    class:bg-slate-700={week === null}
                  ></div></td
                >
              {/each}
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  </div>
</div>
