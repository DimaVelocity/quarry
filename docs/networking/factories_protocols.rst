Factories and Protocols
=======================

Factories
---------

Factories represent your minecraft server or client as a whole. Normally
only one factory is created.

.. module:: quarry.net.client

Client factories require a :class:`~quarry.auth.Profile` object to be supplied
to the initializer. Use the :meth:`ClientFactory.connect` method to connect.
Without a ``protocol_version`` supplied, this method will make two connections
to the server; the first is used to establish the server's protocol version.

.. autoclass:: ClientFactory
    :undoc-members:
    :members: protocol, __init__, connect

.. module:: quarry.net.server

Server factories are used to customize server-wide behaviour. Use
:meth:`~ServerFactory.listen` to listen for connections. A set of all
associated :class:`ServerProtocol` objects is available as
:meth:`~ServerFactory.players`.

.. autoclass:: ServerFactory
    :undoc-members:
    :members: protocol, force_protocol_version, compression_threshold,
        auth_timeout, online_mode, max_players, motd, favicon, __init__,
        listen, players

Protocols
---------

.. module:: quarry.net.protocol

Protocols represent a connection to a remote minecraft server or client. For
most common usages, clients have only one protocol active at any given time. In
protocols you can define packet handlers or override methods in order to
respond to events.

.. class:: quarry.net.protocol.Protocol

    Minecraft protocol implementation common to both clients and servers. You
    should not subclass from this class, but rather subclass from one of the
    three classes below.

    The methods/attributes given below relate specifically to quarry; the rest
    are given in the *Connection*, *Authentication* and *Packets* sections
    further on.

    .. autoattribute:: factory
    .. autoattribute:: logger
    .. autoattribute:: tasks

.. autoclass:: quarry.net.server.ServerProtocol
.. autoclass:: quarry.net.client.ClientProtocol
.. autoclass:: quarry.net.client.SpawningClientProtocol

Connection
''''''''''

Override the :meth:`~Protocol.connection_made`,
:meth:`~Protocol.connection_lost` and :meth:`~Protocol.connection_timed_out`
methods to handle connection events. The remote's IP address is available as
the :attr:`~Protocol.remote_addr` attribute.

In servers, :attr:`~quarry.net.server.ServerProtocol.connect_host` stores the
hostname the client reported that it connected to; this can be used to
implement virtual hosting.

.. automethod:: Protocol.connection_made
.. automethod:: Protocol.connection_lost
.. automethod:: Protocol.connection_timed_out
.. autoattribute:: Protocol.remote_addr
.. autoattribute:: quarry.net.server.ServerProtocol.connect_host
.. autoattribute:: quarry.net.server.ServerProtocol.connect_port
.. automethod:: Protocol.close
.. autoattribute:: Protocol.closed


Authentication
''''''''''''''

Override the :meth:`~Protocol.auth_ok` and :meth:`~Protocol.auth_failed`
methods to handle an authentication outcome. In servers, the player's display
name can be obtained as :attr:`~quarry.net.server.ServerProtocol.display_name`,
with :attr:`~quarry.net.server.ServerProtocol.display_name_confirmed` being set
to ``True`` when authentication is successful. In clients, the in-use profile
is available as ``self.factory.profile``.

Override the :meth:`~Protocol.player_joined` and :meth:`~Protocol.player_left`
methods to respond to a player entering "play" mode (via the authentication
process) or quitting the game from "play" mode. You can check the player's
current status via :attr:`~Protocol.in_game`

.. automethod:: Protocol.auth_ok
.. automethod:: Protocol.auth_failed
.. autoattribute:: quarry.net.server.ServerProtocol.display_name
.. autoattribute:: quarry.net.server.ServerProtocol.display_name_confirmed
.. automethod:: Protocol.player_joined
.. automethod:: Protocol.player_left
.. autoattribute:: Protocol.in_game

Packets
'''''''

Call :meth:`~Protocol.send_packet` to send a packet::

    def packet_keep_alive(self, buff):
        # Read the keep alive ID
        identifier = buff.unpack_varint()
        print(identifier)

        # Send a keep alive back
        self.send_packet("keep_alive", self.buff_type.pack_varint(identifier))


To construct the payload, call static methods on
:class:`~quarry.types.buffer.Buffer`. A reference to this class is available as
``self.buff_type``.

To receive a packet, implement a method in your subclass of
:class:`~quarry.net.client.ClientProtocol` or
:class:`~quarry.net.server.ServerProtocol` with a name like
:samp:`packet_{<packet name>}`::

    def packet_update_health(self, buff):
        health = buff.unpack('f')
        food = buff.unpack_varint()
        saturation = buff.unpack('f')

.. seealso:: :doc:`packet_names`.

You are passed a :class:`~quarry.types.buffer.Buffer` instance, which contains
the payload of the packet. If you hook a packet, you should ensure you read the
entire payload.

Packet dispatching can be customized. If you override
:meth:`~Protocol.packet_unhandled` you can handle any packets without a
matching :samp:`packet_{<packet name>}` handler. If you override
:meth:`~Protocol.packet_received`, you can replace the entire
:samp:`packet_{<packet name>}` dispatching.

.. automethod:: Protocol.send_packet
.. autoattribute:: Protocol.buff_type
.. automethod:: Protocol.packet_received
.. automethod:: Protocol.packet_unhandled
.. automethod:: Protocol.dump_packet
.. automethod:: Protocol.log_packet

