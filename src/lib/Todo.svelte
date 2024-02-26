<script>

    import { createEventDispatcher } from 'svelte';
    let tasks = [];
    let newTask = '';
    export let events;

    const dispatch = createEventDispatcher();

    const colors = ['#ea4b07', '#F9692B', '#EA4A67', '#C4515E', '#FF8163'];

    function addTask() {
        if (newTask.trim() !== '') {
            const taskExists = tasks.some(task => task.title.toLowerCase() === newTask.trim().toLowerCase());
            if (!taskExists) {
                const randomColor = colors[Math.floor(Math.random() * colors.length)]; 
                tasks = [...tasks, { id: Date.now(), title: newTask.trim(), color: randomColor }];
                newTask = ''; // Reset input field
            } else {
                console.log('Task with this name already exists.');
            }
        }
    }

    function removeTask(taskId, taskLabel) {
        events = events.filter(event => event.label !== taskLabel);
        dispatch('update', { events });
        tasks = tasks.filter(task => task.id !== taskId);
    }

    function handleDragStart(event, task) {
        event.dataTransfer.setData("application/task", JSON.stringify(task));
    } 

</script>

<style>
    .task {
        padding: 10px;
        margin-bottom: 5px;
        border-radius: 4px;
        position: relative;
    }
    .task button {
        position: absolute;
        top: 5px; /* Adjust top as needed */
        right: 5px; /* Adjust right as needed */
        font-size: 12px; /* Smaller font size for the x button */
        padding: 2px 5px; /* Smaller padding to reduce size */
        line-height: 1; /* Adjust line height to center the text vertically */
        border-radius: 50%; /* Optional: makes the button circular */
        cursor: pointer; /* Changes the cursor to a pointer on hover */
    }

</style>

<div>
    <input type="text" bind:value={newTask} placeholder="Enter a new task" />
    <button on:click={addTask}>Add Task</button>
</div>

{#if tasks.length > 0}
    <ul>
        {#each tasks as task (task.id)}
            <li draggable="true" class="task" on:dragstart={(event) => handleDragStart(event, task)} style="background-color: {task.color};">
                {task.title}
                <button on:click={() => removeTask(task.id, task.title)}>x</button>
            </li>
        {/each}
    </ul>
{:else}
    <p>No tasks yet. Add a task above!</p>
{/if}