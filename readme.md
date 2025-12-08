# Activity Heatmap Svelte Component
A heatmap displaying user activity, styled after the github user activity heatmap, and the anki heatmap extension.

Displays activity for a date as colour graded blocks for that date. Also displays daily average, days active expressed as a percent, longest streak of active days, and current streak of active days.

## How to Include in your project

1. Add the component file to your component folder

2. You will need to create a data store / interface that exports a date string and a number, should look like this;

```
export interface DailyActivity {
        date: string; // date string (YYYY-MM-DD)
        totalMinutes: number; // Total minutes worked that day
    } 
```

Your metric does not have to be totalminutes, but will require editing the file to work with something other than time.
