<script>
    const { attachments, sender, time, text, side } = $props();



    let edited = $derived(text.includes('<This message was edited>'));

    let messageText = $derived(edited
      ?  text.replace('<This message was edited>', '')
      : text
    );

    let oneTimeMessage = $derived(text.includes('You received a view once message. For added privacy, you can only open it on your phone.'));

    let attachmentName = $derived(
      text.match(/<attached: ([\w\d-]+\.\w+)>/)?.at(1)
    );

    let sticker = $derived(attachmentName.includes('STICKER'));

    let emoji = $derived(/^(\s*\p{Emoji}\s*){1,6}$/u.test(text) ? 'emoji' : '');

    // $effect(() => {
    //   console.log(text)
    //   console.log(attachmentName)
    //   console.log(attachments.get(attachmentName))
    // });
</script>

<div class={"message " + side}>
    <div class={"inner " + emoji}>
        {#if side === 'left' && sender}
            <span class="sender">{sender}</span>
        {/if}
        {#if attachmentName}
            {#if sticker}
                <img class="sticker" src={attachments.get(attachmentName)} alt="sticker" />
            {:else}
                <img class="attachment" src={attachments.get(attachmentName)} alt="attachment" />
            {/if}
        {:else}
            {#if oneTimeMessage}
                <span class="one-time">① One time message</span>
            {:else}
                <p class="text">
                    {messageText}
                </p>
            {/if}
        {/if}
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

    .inner:has(.sticker), .emoji {
        background-color: transparent !important;
        font-size: 2rem;
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

    .sticker {
        max-width: 100px;
        max-height: 100px;
    }
</style>
