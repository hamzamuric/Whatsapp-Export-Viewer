<script>
    import { tick } from "svelte";
    import Message from "./lib/Message.svelte";

    let files = $state();
    let attachments = $state(new Map());
    let messages = $state([]);
    let users = $state([]);
    let selectedUser = $state('');


    $effect(() => {
      if (!files) return;
      console.log(files)

      const filesArray = [...files];
      console.log(filesArray)

      const chatFile = filesArray.find(f => f.name === '_chat.txt')

      const chatReader = new FileReader();
      chatReader.onerror = e => console.error(e);
      chatReader.onload = e => {
        const { parsedMessages, parsedUsers } = parseMessages(e.target.result);
        const parsedUsersArray = Array.from(parsedUsers);
        messages = parsedMessages;
        users = parsedUsersArray;
        selectedUser = parsedUsersArray[0];
      };
      chatReader.readAsText(chatFile);
      tick().then(() => {
        document.querySelector('.messages:last-child')?.scrollIntoView();
      })

      const attachmentFiles = filesArray.filter(f => f.name !== '_chat.txt');
      attachmentFiles.forEach(f => {
        const attachmentReader = new FileReader();
        attachmentReader.onerror = e => console.error(e);
        attachmentReader.onload = e => {
          const attachment = e.target.result;
          attachments.set(f.name, attachment);
        };
        attachmentReader.readAsDataURL(f);
      })
    })

    const messageHeaderRegex = /\[(\d+\.\s\d+\.\s\d+\.)\,\s(\d{2}:\d{2}:\d{2})\]\s([^:]+):\s/g;
    function parseMessages(chatText) {
      const matches = Array.from(chatText.matchAll(messageHeaderRegex));
      const parsedMessages = [];
      const parsedUsers = new Set();

      let lastDate = '';
      for (let i in matches) {
        const match = matches[+i];

        const { index } = match;
        const nextIndex = matches[+i + 1]?.index ?? chatText.length;

        let [header, date, time, sender] = match;
        time = time.split(':').slice(0, 2).join(':');
        parsedUsers.add(sender);

        if (date !== lastDate) {
            parsedMessages.push({ sender: 'SYSTEM', date });
            lastDate = date;
        }

        const text = chatText.slice(index + header.length, nextIndex).trim();

        parsedMessages.push({ sender, date, time, text });
      }

      return { parsedMessages, parsedUsers };
    }
</script>

{#if users.length === 0}
    <input type="file" bind:files webkitdirectory directory multiple />
{:else}
    <select bind:value={selectedUser}>
        {#each Array.from(users) as user}
            <option value={user}>{user}</option>
        {/each}
    </select>
    <div class="messages">
        {#each messages as message}
            {#if message.sender === 'SYSTEM'}
                <div class="new-date"><span>{message.date}</span></div>
            {:else}
                <Message {...message} attachments={attachments} sender={users.length > 2 ? message.sender : undefined} side={message.sender === selectedUser ? 'right' : 'left'} />
            {/if}
        {/each}
    </div>
{/if}

<style>
    .messages {
        display: flex;
        flex-direction: column;
        gap: 12px;
        overflow-y: scroll;
        max-height: 90vh;
    }

    .new-date {
        display: flex;
        justify-content: center;
    }

    .new-date span {
        background-color: lightgray;
        padding: 2px 6px;
        border-radius: 8px;
        font-size: 0.7rem;
        color: black;
        opacity: 0.8;
    }
</style>
