# Discord RPC

This code in written in pyhton and can be used by anyone by makeing a few changes and it will be all good to go. 




> What is the use - this can be used to customize yor status in discord like this 
> ![](C:\Users\chatt\AppData\Roaming\marktext\images\2021-06-15-12-11-51-image.png)
> 
> 
> 
> 
> 

## What you need?

just python 



### What need to be changed?

![](https://raw.githubusercontent.com/niveshbirangal/discord-rpc/master/readmeassets/createapp.gif)



![](https://github.com/niveshbirangal/discord-rpc/raw/master/readmeassets/selectimage.png)

Select images and clear all fields except `PARTY ID` and `JOIN SECRET`

![](https://github.com/niveshbirangal/discord-rpc/raw/master/readmeassets/fileds.png)

Now go to rpc.py, put your client id, and change the activity code according to your best fit


![](C:\Users\chatt\AppData\Roaming\marktext\images\2021-06-15-12-15-50-image.png)

![](C:\Users\chatt\AppData\Roaming\marktext\images\2021-06-15-12-16-38-image.png)

Make sure your desktop app is running and then run the file

![](C:\Users\chatt\AppData\Roaming\marktext\images\2021-06-15-12-17-17-image.png)

![](C:\Users\chatt\AppData\Roaming\marktext\images\2021-06-15-12-17-30-image.png)



# The python code (raw)

```python
import asyncio
import json
import os
import struct
import sys
import time
import uuid


class DiscordRPC:
    def __init__(self):
        if sys.platform == 'linux' or sys.platform == 'darwin':
            env_vars = ['XDG_RUNTIME_DIR', 'TMPDIR', 'TMP', 'TEMP']
            path = next((os.environ.get(path, None) for path in env_vars if path in os.environ), '/tmp')
            self.ipc_path = f'{path}/discord-ipc-0'
            self.loop = asyncio.get_event_loop()
        elif sys.platform == 'win32':
            self.ipc_path = r'\\?\pipe\discord-ipc-0'
            self.loop = asyncio.ProactorEventLoop()

        self.sock_reader: asyncio.StreamReader = None
        self.sock_writer: asyncio.StreamWriter = None

    async def read_output(self):
        while True:
            data = await self.sock_reader.read(1024)
            if data == b'':
                self.sock_writer.close()
                exit(0)
            try:
                code, length = struct.unpack('<ii', data[:8])
                print(f'OP Code: {code}; Length: {length}\nResponse:\n{json.loads(data[8:].decode("utf-8"))}\n')
            except struct.error:
                print(f'Something happened\n{data}')

    def send_data(self, op: int, payload: dict):
        payload = json.dumps(payload)
        self.sock_writer.write(struct.pack('<ii', op, len(payload)) + payload.encode('utf-8'))

    async def handshake(self):
        if sys.platform == 'linux' or sys.platform == 'darwin':
            self.sock_reader, self.sock_writer = await asyncio.open_unix_connection(self.ipc_path, loop=self.loop)
        elif sys.platform == 'win32':
            self.sock_reader = asyncio.StreamReader(loop=self.loop)
            reader_protocol = asyncio.StreamReaderProtocol(self.sock_reader, loop=self.loop)
            self.sock_writer, _ = await self.loop.create_pipe_connection(lambda: reader_protocol, self.ipc_path)

        self.send_data(0, {'v': 1, 'client_id': '854235041420541953'})
        data = await self.sock_reader.read(1024)
        code, length = struct.unpack('<ii', data[:8])
        print(f'OP Code: {code}; Length: {length}\nResponse:\n{json.loads(data[8:].decode("utf-8"))}\n')
    
    def send_rich_presence(self):
        current_time = time.time()
        payload = {
            'cmd': 'SET_ACTIVITY',
            'args': {
                'activity': {
                    'state': 'Visual Studio Code',
                    'details': 'Not Somthing You Can Process',
                    'timestamps': {
                        'start': int(current_time),
                        'end': int(current_time) + (100 * 60)
                        
                    },
                    'assets': {
                        'large_text': '>tfw no gf',
                        'large_image': 'vsc-logo',
                        'small_text': 'that\'s me',
                        'small_image': 'gio'
                    },
                    'party': {
                        'id': 'vsc',
                        'size': [1, 1]  # [Minimum, Maximum]
                    },
                    'secrets': {
                        'match': 'install_csc',
                        'join': 'communism_is_bad',
                        'spectate': 'b0nzybuddy',
                    },
                    'instance': True
                },
                'pid': os.getpid()
            },
            'nonce': str(uuid.uuid4())
        }
        self.send_data(1, payload)

    async def run(self):
        await self.handshake()
        self.send_rich_presence()
        await self.read_output()

    def close(self):
        self.sock_writer.close()
        self.loop.close()
        exit(0)


if __name__ == '__main__':
    rpc = DiscordRPC()
    try:
        rpc.loop.run_until_complete(rpc.run())
    except KeyboardInterrupt:
        rpc.close()

        
        print("Made by")
```
