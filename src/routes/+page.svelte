<script>
    import ActivityHeatmap from '$lib/components/ActivityHeatmap.svelte';
    import { addTimeToDate, dailyActivity } from '../stores/dailyActivity';

    let selectedDate = new Date().toISOString().split('T')[0]; // Today's date
    let minutesToAdd = 25;
</script>

<div class="demo-controls">
    <h2>Add Activity</h2>

    <label>
        Date:
        <input type="date" bind:value={selectedDate} max={new Date().toISOString().split('T')[0]} />
    </label>

    <label>
        Minutes:
        <input type="number" bind:value={minutesToAdd} min="1" />
    </label>

    <button on:click={() => addTimeToDate(selectedDate, minutesToAdd)}>
        Add {minutesToAdd} minutes
    </button>

    <div class="quick-buttons">
        <button on:click={() => addTimeToDate(selectedDate, 15)}>+15 min</button>
        <button on:click={() => addTimeToDate(selectedDate, 25)}>+25 min</button>
        <button on:click={() => addTimeToDate(selectedDate, 50)}>+50 min</button>
        <button on:click={() => addTimeToDate(selectedDate, 90)}>+90 min</button>
    </div>

    <!-- Show current total for selected date -->
    <p>
        Total for {selectedDate}:
        <strong>{$dailyActivity.find(a => a.date === selectedDate)?.totalMinutes || 0} minutes</strong>
    </p>

    <!-- Clear all button for testing -->
    <button class="clear" on:click={() => dailyActivity.set([])}>
        Clear All Data
    </button>
</div>

<ActivityHeatmap activityStore={dailyActivity} />

<style>
    .demo-controls {
        padding: 20px;
        background: #f5f5f5;
        border-radius: 8px;
        margin-bottom: 20px;
    }

    label {
        display: block;
        margin: 10px 0;
    }

    input {
        margin-left: 10px;
        padding: 5px;
    }

    button {
        padding: 8px 16px;
        margin: 5px;
        cursor: pointer;
        background: #4CAF50;
        color: white;
        border: none;
        border-radius: 4px;
    }

    button:hover {
        background: #45a049;
    }

    button.clear {
        background: #f44336;
        margin-top: 10px;
    }

    button.clear:hover {
        background: #da190b;
    }

    .quick-buttons {
        margin-top: 10px;
    }
</style>