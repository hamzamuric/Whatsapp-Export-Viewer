<script>
    const { sender, time, text, side, emoji, edited } = $props();
</script>

<div class={"message " + side}>
    <div class={"inner " + (emoji ? 'emoji' : '')}>
        {#if side === 'left' && sender}
            <span class="sender">{sender}</span>
        {/if}
        <p class="text">
            {text}
        </p>
    </div>
    <div class="meta">
        <span class="time">{time}</span>
        {#if edited}
            <span class="edited">✎ Edited</span>
        {/if}
    </div>
</div>

<style>
    .message {
        margin: 0 20px;
        padding: 0 12px;
        color: white;
        display: flex;
        align-items: center;
        gap: 12px;
    }

    .right {
        flex-direction: row-reverse;
    }

    .inner {
        max-width: 60ch;
        display: inline-block;
        padding: 4px 8px;
        display: flex;
        flex-direction: column;
        border-radius: 12px;
    }

    .left .inner {
        background-color: grey;
        justify-content: flex-start;
        float: left;
        border-bottom-left-radius: 0;
    }

    .right .inner {
        background-color: green;
        justify-content: flex-end;
        float: right;
        border-bottom-right-radius: 0;
    }

    .sender {
        font-weight: bold;
        font-size: 0.9rem;
    }

    .left .sender, .left .text {
        text-align: left;
    }

    .right .sender, .right .text {
        text-align: right;
    }

    .text {
        display: flex;
        justify-content: space-between;
        align-items: baseline;
        gap: 6px;
        font-family: Verdana, Geneva, Tahoma, sans-serif;
        overflow-wrap: break-word;
        overflow: clip;
    }

    .emoji>.text {
        overflow: unset;
    }

    .inner:has(.sticker), .emoji {
        background-color: transparent !important;
        font-size: 2rem;
    }

    .time {
        font-size: 0.6rem;
        font-weight: bold;
        opacity: 0.6;
    }

    .left .text, .left .time {
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;
    }

    .right .text, .right .time {
        display: flex;
        flex-direction: row-reverse;
        flex-wrap: wrap;
    }

    .meta {
        display: flex;
        flex-direction: column;
    }

    .meta span {
        line-height: 1.2;
    }

    .edited {
        font-size: 0.6rem;
        font-weight: bold;
        opacity: 0.6;
    }
</style>
