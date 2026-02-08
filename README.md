# SerialTransfer.NET

A C# port of the [Arduino SerialTransfer](https://github.com/PowerBroker2/SerialTransfer) library for robust, packetized serial communication between .NET applications and Arduino devices.

## Overview

SerialTransfer.NET enables reliable communication between C# applications and Arduino devices using a packet-based protocol with CRC validation. It supports sending both text messages and structured binary data with minimal overhead.

Key features:
- Packet framing with start/stop bytes
- CRC error checking
- Support for different packet types via packet IDs
- Transmitting and receiving structured data
- Automatic packet acknowledgment

## Installation

### Requirements

- .NET Framework 4.6.2+ or .NET Core 3.1+ / .NET 5.0+
- System.IO.Ports package for SerialPort functionality

### Via NuGet (Coming Soon)

```
Install-Package SerialTransfer.NET
```

### Manual Installation

1. Clone this repository
2. Add the project to your solution or directly reference the DLL

## Project Structure

```
SerialTransfer.NET/
├── src/                     # Source code for the library
│   ├── PacketCRC.cs         # CRC implementation
│   ├── Packet.cs            # Packet handling
│   └── SerialTransfer.cs    # Main SerialTransfer class
├── samples/                 # Example applications
└── tests/                   # Unit and integration tests
```

## Usage

### Basic Example

```csharp
using System;
using System.IO.Ports;
using System.Text;
using SerialTransfer;

// Create and configure SerialPort
SerialPort port = new SerialPort
{
    PortName = "COM3",
    BaudRate = 115200,
    DataBits = 8,
    Parity = Parity.None,
    StopBits = StopBits.One
};

// Create SerialTransfer instance
SerialTransfer transfer = new SerialTransfer();
transfer.Begin(port);

// Send text message
string message = "Hello Arduino!";
byte[] messageBytes = Encoding.ASCII.GetBytes(message);

for (int i = 0; i < messageBytes.Length; i++)
{
    transfer.Packet.TxBuff[i] = messageBytes[i];
}

transfer.SendData((ushort)messageBytes.Length, 1);

// Receive response
if (transfer.Available() > 0)
{
    byte packetId = transfer.CurrentPacketId();
    byte bytesRead = transfer.BytesRead;
    
    string response = Encoding.ASCII.GetString(
        transfer.Packet.RxBuff, 0, bytesRead);
    
    Console.WriteLine($"Response: {response}");
}

// Clean up
transfer.Close();
```

### Sending and Receiving Structs

```csharp
// Define a struct for binary transmission
[StructLayout(LayoutKind.Sequential, Pack = 1)]
struct SensorData
{
    public int Id;
    public float Temperature;
    public float Humidity;
    public byte Status;
}

// Create data to send
SensorData data = new SensorData
{
    Id = 42,
    Temperature = 23.5f,
    Humidity = 65.0f,
    Status = 1
};

// Manually serialize and send the struct
int structSize = Marshal.SizeOf(data);
byte[] structBytes = new byte[structSize];

GCHandle handle = GCHandle.Alloc(structBytes, GCHandleType.Pinned);
try
{
    Marshal.StructureToPtr(data, handle.AddrOfPinnedObject(), false);
}
finally
{
    handle.Free();
}

// Copy to transmit buffer
for (int i = 0; i < structBytes.Length; i++)
{
    transfer.Packet.TxBuff[i] = structBytes[i];
}

// Send with packet ID 2
transfer.SendData((ushort)structSize, 2);

// Receive and deserialize struct
if (transfer.Available() > 0)
{
    SensorData receivedData = new SensorData();
    transfer.RxObj(ref receivedData);
    
    Console.WriteLine($"Received: ID={receivedData.Id}, " +
                      $"Temp={receivedData.Temperature}°C");
}
```

## Arduino Compatibility

This library is designed to work with the Arduino SerialTransfer library. Your Arduino sketch should include:

```cpp
#include "SerialTransfer.h"

SerialTransfer myTransfer;

void setup() {
  Serial.begin(115200);
  myTransfer.begin(Serial);
}

void loop() {
  if(myTransfer.available()) {
    // Process received data
    // ...
    
    // Send response
    // ...
  }
}
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- This project is a C# port of the [Arduino SerialTransfer library](https://github.com/PowerBroker2/SerialTransfer)
- Thanks to the original author for the protocol design