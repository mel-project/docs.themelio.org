# Network protocols

This section contains specifications for all the network protocols used in Themelio.

Themelio's protocols can be widely divided into **internal** and **client-facing** protocols. Internal protocols are typically RLP-based and are overlaid on [libp2p](https://github.com/libp2p/); stakeholder and auditor nodes run the internal protocols to synchronize state. Client-facing protocols use RLP over HTTP and are used by clients to communicate with auditors. 

