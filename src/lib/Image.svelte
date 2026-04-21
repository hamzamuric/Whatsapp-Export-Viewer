<script>
    const { attachment, side, sender, time } = $props();
</script>

<div class={"message " + side}>
    <div class="inner">
        {#if side === 'left' && sender}
            <span class="sender">{sender}</span>
        {/if}
        <img class="attachment" src={attachment} alt="attachment" />
    </div>
    <div class="meta">
        <span class="time">{time}</span>
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

    .text {
        display: flex;
        justify-content: space-between;
        align-items: baseline;
        gap: 6px;
        font-family: Verdana, Geneva, Tahoma, sans-serif;
    }

    .time {
        font-size: 0.6rem;
        font-weight: bold;
        opacity: 0.6;
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

    .one-time {
        opacity: 0.8;
        -webkit-touch-callout: none;
        -webkit-user-select: none;
        -khtml-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
    }

    .attachment {
        max-width: 60ch;
        max-height: 400px;
        border-radius: 12px;
        padding: 6px 0;
    }
</style>
