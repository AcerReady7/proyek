## Restore deleted Telegram messages, medias and files from groups

There's not telegram API method for this, we need to call MTProto methods to retrieve messages from the "Recent Actions" (Admin Log) since deleted messages (and medias) gets moved there for 48 hours before the permanent deletion.

```python

from telethon import TelegramClient, events, sync
from telethon.tl.types import InputChannel, PeerChannel
from telethon.tl.types import Channel
import time

# Get your own api_id and
# api_hash from https://my.telegram.org, under API Development.
api_id = API_ID
api_hash = API_HASH

client = TelegramClient('session_name', api_id, api_hash)
client.start()

group = client.get_entity(PeerChannel(GROUP_CHAT_ID))

#messages = client.get_admin_log(group)

file1 = open("dump.json","w") 
c = 0
m = 0
for event in client.iter_admin_log(group):
    if event.deleted_message:
        print("Dumping message",c, "(", event.old.id, event.old.date,")")
        file1.write(event.old.to_json() + ",") 
        c+=1
        if event.old.media:
            m+=1
            #print(event.old.media.to_dict()['Document']['id'])
            client.download_media(event.old.media, str(event.old.id))
            print(" Dumped media", m)
        time.sleep(0.1)
```