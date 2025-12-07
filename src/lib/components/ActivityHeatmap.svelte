<script lang="ts">
    import type { DailyActivity } from './stores/dailyActivity';
    import type { Writable } from 'svelte/store';

    export let activityStore: Writable<DailyActivity[]>;

    interface DayCell {
        date: Date;
        dateString: string;
        minutes: number;
        intensity: number; // 0-4 scale for color intensity
    }

    let activities: DailyActivity[] = [];
    let grid: DayCell[][] = [];
    let months: Array<{ name: string; column: number }> = [];
    let maxMinutes = 0;

    $: activities = $activityStore;
    $: if (activities) {
        generateGrid();
    }

    function generateGrid() {
        if (!activities || activities.length === 0) {
            grid = Array.from({ length: 7 }, () => []);
            months = [];
            maxMinutes = 0;
            // Still generate empty grid for structure
        }

        const today = new Date();
        const endDate = new Date(today);

        // Go back ~52 weeks to show a full year
        const startDate = new Date(today);
        startDate.setDate(startDate.getDate() - 364);

        // Adjust to start on Sunday
        const dayOfWeek = startDate.getDay();
        startDate.setDate(startDate.getDate() - dayOfWeek);

        // Create activity map for quick lookup
        const activityMap = new Map<string, number>();
        if (activities) {
            activities.forEach(a => {
                activityMap.set(a.date, a.totalMinutes);
            });
        }

        // Calculate max minutes for scaling
        maxMinutes = activities && activities.length > 0 ? Math.max(...activities.map(a => a.totalMinutes), 1) : 1;

        // Generate grid (7 rows for days of week, columns for weeks)
        const tempGrid: DayCell[][] = Array.from({ length: 7 }, () => []);
        const monthsTemp: Array<{ name: string; column: number }> = [];
        let currentMonth = -1;

        const currentDate = new Date(startDate);
        let weekIndex = 0;

        while (currentDate <= today) {
            const dateString = currentDate.toISOString().split('T')[0];
            const minutes = activityMap.get(dateString) || 0;
            const intensity = minutes > 0 ? Math.min(4, Math.ceil((minutes / maxMinutes) * 4)) : 0;

            const dayOfWeek = currentDate.getDay();

            tempGrid[dayOfWeek].push({
                date: new Date(currentDate),
                dateString,
                minutes,
                intensity
            });

            // Track month changes for labels
            if (currentDate.getMonth() !== currentMonth && dayOfWeek === 0) {
                currentMonth = currentDate.getMonth();
                monthsTemp.push({
                    name: currentDate.toLocaleDateString('en-US', { month: 'short' }),
                    column: weekIndex
                });
            }

            // Move to next day
            currentDate.setDate(currentDate.getDate() + 1);
            if (currentDate.getDay() === 0 && currentDate <= today) {
                weekIndex++;
            }
        }

        grid = tempGrid;
        months = monthsTemp;
    }

    function getColor(intensity: number): string {
        const colors = [
            '#ebedf0', // 0 - no activity (grey)
            '#9be9a8', // 1 - low
            '#40c463', // 2 - medium-low
            '#30a14e', // 3 - medium-high
            '#216e39'  // 4 - high
        ];
        return colors[intensity] || colors[0];
    }

    function formatTooltip(cell: DayCell): string {
        const dateStr = cell.date.toLocaleDateString('en-US', {
            weekday: 'short',
            year: 'numeric',
            month: 'short',
            day: 'numeric'
        });

        if (cell.minutes === 0) {
            return `${dateStr}: No activity`;
        }

        const hours = Math.floor(cell.minutes / 60);
        const mins = cell.minutes % 60;

        if (hours > 0) {
            return `${dateStr}: ${hours}h ${mins}m`;
        } else {
            return `${dateStr}: ${mins}m`;
        }
    }

    const dayLabels = ['S', 'M', 'T', 'W', 'T', 'F', 'S'];
</script>

<div class="heatmap-container">
    <div class="heatmap">
        <!-- Day labels -->
        <div class="day-labels">
            {#each dayLabels as label}
                <div class="day-label">{label}</div>
            {/each}
        </div>

        <div class="grid-wrapper">
            <!-- Month labels -->
            <div class="month-labels">
                {#each months as month}
                    <div class="month-label" style="left: {month.column * 13}px">
                        {month.name}
                    </div>
                {/each}
            </div>

            <!-- Grid -->
            <div class="grid">
                {#each Array(7) as _, dayIndex}
                    <div class="grid-row">
                        {#each grid[dayIndex] || [] as cell}
                            <div
                                    class="cell"
                                    class:empty={cell.minutes === 0}
                                    style="background-color: {getColor(cell.intensity)}"
                                    title={formatTooltip(cell)}
                            ></div>
                        {/each}
                    </div>
                {/each}
            </div>
        </div>
    </div>

    <!-- Legend -->
    <div class="legend">
        <span class="legend-label">Less</span>
        {#each [0, 1, 2, 3, 4] as intensity}
            <div
                    class="legend-cell"
                    style="background-color: {getColor(intensity)}"
            ></div>
        {/each}
        <span class="legend-label">More</span>
    </div>
</div>

<style>
    .heatmap-container {
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell', sans-serif;
        padding: 20px;
        max-width: 100%;
    }

    .heatmap {
        display: flex;
        gap: 4px;
    }

    .day-labels {
        display: flex;
        flex-direction: column;
        gap: 3px;
        padding-top: 24px;
        margin-right: 8px;
    }

    .day-label {
        font-size: 10px;
        color: #666;
        height: 10px;
        line-height: 10px;
        text-align: right;
    }

    .day-label-spacer {
        height: 10px;
    }

    .grid-wrapper {
        position: relative;
        flex: 1;
    }

    .month-labels {
        position: absolute;
        top: 0;
        left: 0;
        height: 20px;
        width: 100%;
    }

    .month-label {
        position: absolute;
        font-size: 10px;
        color: #666;
    }

    .grid {
        display: flex;
        flex-direction: column;
        gap: 3px;
        padding-top: 24px;
    }

    .grid-row {
        display: flex;
        flex-direction: row;
        gap: 3px;
    }

    .cell {
        width: 10px;
        height: 10px;
        border-radius: 2px;
        cursor: pointer;
        transition: all 0.1s ease;
    }

    .cell:hover {
        outline: 2px solid rgba(0, 0, 0, 0.3);
        outline-offset: 1px;
        transform: scale(1.1);
    }

    .cell.empty {
        opacity: 0.8;
    }

    .legend {
        display: flex;
        align-items: center;
        gap: 4px;
        margin-top: 16px;
        justify-content: flex-end;
        font-size: 11px;
        color: #666;
    }

    .legend-label {
        margin: 0 4px;
    }

    .legend-cell {
        width: 10px;
        height: 10px;
        border-radius: 2px;
    }
</style>