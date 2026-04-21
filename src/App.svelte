<script>
    import { tick } from "svelte";
    import Message from "./lib/Message.svelte";
    import Sticker from "./lib/Sticker.svelte";
    import OneTime from "./lib/OneTime.svelte";
    import Audio from "./lib/Audio.svelte";
    import Video from "./lib/Video.svelte";
    import Image from "./lib/Image.svelte";

    let files = $state();
    let attachments = $state(new Map());
    let messages = $state([]);
    let users = $state([]);
    let selectedUser = $state('');
    let attachmentsLoaded = $state([false]);
    let loaded = $derived(attachmentsLoaded.every(loaded => loaded));
    let filesArray = $state([]);


    $effect(() => {
      if (!files) return;
      if (filesArray.length > 0) return;

      filesArray = [...files];


      const attachmentFiles = filesArray.filter(f => f.name !== '_chat.txt');

      attachmentsLoaded = attachmentFiles.map(_ => false);

      attachmentFiles.forEach((f, i) => {
        const attachmentReader = new FileReader();
        attachmentReader.onerror = e => console.error(e);
        attachmentReader.onload = e => {
          const attachment = e.target.result;
          attachments.set(f.name, attachment);
          attachmentsLoaded[i] = true;
        };
        attachmentReader.readAsDataURL(f);
      })
    })

    $effect(() => {
      if (selectedUser) return;
      if (loaded) {
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
      }
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
            parsedMessages.push({ type: 'system',  date });
            lastDate = date;
        }

        const text = chatText.slice(index + header.length, nextIndex).trim();

        let attachmentName = text.match(/<attached: ([\w\d-]+\.\w+)>/)?.at(1);

        if (attachmentName) {
          const attachment = attachments.get(attachmentName);
          const parts = attachmentName.split('.');
          const extension = parts.at(-1);

          if (['jpg', 'jpeg', 'png', 'gif', 'webp'].includes(extension)) {
            if (attachmentName.includes('STICKER')) {
              parsedMessages.push({ type: 'sticker', sender, time, sticker: attachment });
            } else {
              parsedMessages.push({ type: 'image', sender, time, attachment });
            }
          } else if (['mp4', 'mov', 'avi', 'mkv'].includes(extension)) {
            parsedMessages.push({ type: 'video', format: extension, sender, time, attachment });
          } else if (['opus', 'wav', 'ogg', 'mp3'].includes(extension)) {
            parsedMessages.push({ type: 'audio', sender, time, attachment });
          }


          continue;
        }

        let oneTimeMessage = text.includes('You received a view once message. For added privacy, you can only open it on your phone.');
        if (oneTimeMessage) {
          parsedMessages.push({ type: 'one-time', sender, time, text });
          continue;
        }


        const edited = text.includes('<This message was edited>');

        let messageText = edited
          ?  text.replace('<This message was edited>', '')
          : text;


        let emoji = /^(\s*\p{Emoji}\s*){1,6}$/u.test(text) && !/\d+/.test(text);

        parsedMessages.push({ type: 'text', emoji, sender, time, text: messageText, edited });
      }

      return { parsedMessages, parsedUsers };
    }
</script>

<div id="app">
{#if users.length === 0}
    <input type="file" bind:files webkitdirectory /* directory */ multiple />
{:else if !loaded}
    <div>Loading...</div>
{:else}
    <select bind:value={selectedUser}>
        {#each Array.from(users) as user}
            <option value={user}>{user}</option>
        {/each}
    </select>
    <div class="messages">
        {#each messages as message}
            {#if message.type === 'system'}
                <div class="new-date"><span>{message.date}</span></div>
            {:else if message.type === 'sticker'}
                <Sticker time={message.time} sticker={message.sticker} sender={users.length > 2 ? message.sender : undefined} side={message.sender === selectedUser ? 'right' : 'left'} />
            {:else if message.type === 'audio'}
                <Audio time={message.time} attachment={message.attachment} sender={users.length > 2 ? message.sender : undefined} side={message.sender === selectedUser ? 'right' : 'left'} />
            {:else if message.type === 'video'}
                <Video time={message.time} attachment={message.attachment} format={message.format} sender={users.length > 2 ? message.sender : undefined} side={message.sender === selectedUser ? 'right' : 'left'} />
            {:else if message.type === 'image'}
                <Image time={message.time} attachment={message.attachment} sender={users.length > 2 ? message.sender : undefined} side={message.sender === selectedUser ? 'right' : 'left'} />
            {:else if message.type === 'one-time'}
                <OneTime time={message.time} sender={users.length > 2 ? message.sender : undefined} side={message.sender === selectedUser ? 'right' : 'left'} />
            {:else}
                <Message text={message.text} time={message.time} edited={message.edited} emoji={message.emoji} sender={users.length > 2 ? message.sender : undefined} side={message.sender === selectedUser ? 'right' : 'left'} />
            {/if}
        {/each}
    </div>
{/if}
</div>

<style>
    :global(body) {
        display: grid;
        place-items: center;
        background-color: darkslategray;
    }

    #app {
        max-height: 85vh;
    }

    .messages {
        display: flex;
        flex-direction: column;
        gap: 12px;
        overflow-y: scroll;
    }

    .new-date {
        display: flex;
        justify-content: center;
    }

    .new-date span {
        background-color: gray;
        padding: 2px 6px;
        border-radius: 8px;
        font-size: 0.7rem;
        color: white;
        opacity: 0.6;
    }

    input[type="file"] {
        padding: 6px 12px;
        color: white;
    }

    select {
        padding: 6px 12px;
    }
</style>
