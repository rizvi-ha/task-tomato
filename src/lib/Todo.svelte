<script>
    let tasks = [];
    let newTask = '';

    function addTask() {
        if (newTask.trim() !== '') {
            tasks = [...tasks, { id: Date.now(), title: newTask, completed: false }];
            newTask = ''; // Reset input field
        }
    }

    function removeTask(taskId) {
        tasks = tasks.filter(task => task.id !== taskId);
    }
</script>

<style>
    .task {
        background-color: #ea4b07;
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
            <li class="task">
                {task.title}
                <button on:click={() => removeTask(task.id)}>x</button>
            </li>
        {/each}
    </ul>
{:else}
    <p>No tasks yet. Add a task above!</p>
{/if}