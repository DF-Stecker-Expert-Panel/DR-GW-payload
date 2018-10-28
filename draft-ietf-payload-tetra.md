%%%

    #
    # Generation tool chain:
    #   mmark (https://github.com/miekg/mmark)
    #   xml2rfc (http://xml2rfc.ietf.org/)
    #   docker container of paulej/rfctools

    Title = "RTP Payload Format for the TETRA Audio Codec"
    abbrev = "RTP Payload Format for TETRA Audio codec"
    category = "std"
    docName = "draft-ietf-payload-tetra-01"
    ipr= "trust200902"
    area = "Internet"
    date = 2018-10-23T12:00:00Z
    workgroup = "payload"

    [pi]
    subcompact = "yes"

    [[author]]
    surname = "Reisenbauer"
    fullname = "Andreas Reisenbauer"
    organization = "Frequentis AG"
    abbrev = "Frequentis"
      [author.address]
      email = "andreas.reisenbauer@frequentis.com"
      [author.address.postal]
      street = "Innovationsstr. 1"
      city = "Vienna"
      code = "1100"
      country = "Austria"

    [[author]]
    surname = "Brandhuber"
    fullname = "Udo Brandhuber"
    organization = "eurofunk Kappacher GmbH"
    abbrev = "eurofunk"
      [author.address]
      email = "ubrandhuber@eurofunk.com"
      [author.address.postal]
      country = "Germany"

    [[author]]
    surname = "Hagedorn"
    fullname = "Joachim Hagedorn"
    organization = "Hagedorn Informationssysteme GmbH"
    abbrev = "Hagedorn"
      [author.address]
      email = "joachim@hagedorn-infosysteme.de"
      [author.address.postal]
      country = "Germany"

    [[author]]
    surname = "Höhnsch"
    fullname = "Klaus-Peter Höhnsch"
    organization = "T-Systems International GmbH"
    abbrev = "T-Systems"
      [author.address]
      email = "klaus-peter.hoehnsch@t-systems.com"
      [author.address.postal]
      country = "Germany"

    [[author]]
    surname = "Wenk"
    fullname = "Stefan Wenk"
    organization = "Frequentis AG"
    abbrev = "Frequentis"
      [author.address]
      email = "stefan.wenk@frequentis.com"
      [author.address.postal]
      street = "Innovationsstr. 1"
      city = "Vienna"
      code = "1100"
      country = "Austria"

    #
    # Revision History
    #   00 - Initial draft.
    #

%%%

.# Abstract

This document specifies a Real-time Transport Protocol (RTP) payload format for TETRA encoded speech signals. The payload format is designed to be able to interoperate with existing TETRA transport formats on non-IP networks. A media type registration is included, specifying the use of the RTP payload format and the storage format.

{mainmatter}

# Introduction

This document specifies the payload format for packetization of TErrestial Trunked RAdio (TETRA) encoded speech signals [@!ETSI-TETRA-Codec] into the Real-time Transport Protocol (RTP) [@!RFC3550]. The payload format supports transmission of multiple channels, multiple frames per payload, robustness against packet loss, and interoperation with existing TETRA transport formats on non-IP networks, as described in Section [](#MediaFormatBackground).

The payload format itself is specified in Section [](#PayloadFormat). 

# Conventions Used In This Document

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document are to be interpreted as described in [@!RFC2119] when they appear in ALL CAPS. These words may also appear in this document in lower case as plain English words, absent their normative meanings.

The following acronyms are used in this document:

* ETSI: European Telecommunications Standards Institute
* TETRA: TErrestial Trunked RAdio

The byte order used in this document is network byte order, i.e., the most significant byte first. The bit order is also the most significant bit first. This is presented in all figures as having the most significant bit leftmost on a line and with the lowest number. Some bit fields may wrap over multiple lines in which cases the bits on the first line are more significant than the bits on the next line.

Best current practices for writing an RTP payload format specification were followed [@RFC2736] updated with [@RFC8088].

# Media Format Background {#MediaFormatBackground}
The TETRA codec is used as vocoder for TETRA systems. The TETRA codec is designed for compressing 30ms of audio speech data into 137 bits. The TETRA codec is designed in such a way that on the air interface two of these 30ms samples are transported together (sub-block 1 and sub-block 2). The codec allows that data of the first 30ms voice frame can be stolen and used for other purposes, e.g. for the exchange of dynamically updated key-material in end-to-end encrypted voice sessions. Codec payload serialisation is specified for TDM lines with 2048 kBit/s within traditional circuit mode based TETRA system. For this purpose two optional formats are defined [@!ETSI-TETRA-Codec], the first format is called FSTE (First Speech Transport Encoding Format), the other format is called OSTE (Optimized Speech Transport Encoding Format). These two formats differ mainly insofar that the OSTE format transports an additional 5 bit frame number, which provides timing information from the air interface to the receiving side in order to save the need for buffering due to different transports speed on air and in 64 kbit/s circuit switched networks. The RTP payload format is defined such that the value of this frame number can be transported.

# Payload format {#PayloadFormat}
The RTP payload format is designed in such a way that it can carry the information needed to map the FSTE and OSTE format from [@!ETSI-TETRA-ISI]. The RTP format is defined such that both of the independent sub-blocks can be transferred separately or together within one RTP packet. Both of them contain the same information in terms of control bits - the information is propagated redundantly. This redundancy is driven by on one hand to simplify the encoding process in direction from E1 to RTP on the other to provide the option to go for either 30ms or 60ms packet size. The redundant information  **SHALL** be propagated consistently equal - otherwise the behavior of the receiver is unspecified.
The payload format is chosen such that the TETRA data bits are octet aligned. 

## RTP Header Usage

The format of the RTP header is specified in [@!RFC3550].  The use of the fields of the RTP header by the TETRA payload format is consistent with that specification.

The payload length of TETRA is an integer number of octets; therefore, no padding is necessary.

The timestamp, sequence number, and marker bit (M) of the RTP header are used in accordance with Section 4.1 of [@!RFC3551].

The RTP payload type for Tetra is to be assigned dynamically.

## Payload layout

RTP payload is composed of multiple blocks with TETRA audio data. TETRA Audio data itself contains:
  - Audio Payload Header
  - Audio Data (137 Bit)
  - 7 Spare Bits

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |I|F|  CTRL   |C|FRAME_NR |  R  |D(1)                           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                           D(137)|  S          |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

RTP payload can be formed by any integer multiple of 30ms audio using following layout (e.g. 90ms audio payload):

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |I|F|  CTRL   |C|FRAME_NR |  R  |D(1)                           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                           D(137)|  S          |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |I|F|  CTRL   |C|FRAME_NR |  R  |D(1)                           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                           D(137)|  S          |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |I|F|  CTRL   |C|FRAME_NR |  R  |D(1)                           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                           D(137)|  S          |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+


## Payload Header

### I bit: Frame Indicator

1: The following frame contains a first block of two sub-blocks

0: The following frame contains a separated sub-block. A sub-block marked as such could either be a second sub-block, or an independent block, which does not have a relation with any first block. To distinguish between the one and the other the information of the Control bits has to be evaluated.

### F bit: Frame Type


Value|Frame contains
-----|-----------------
  0  |FSTE encoded data
  1  |OSTE encoded data


### CTRL: Control bit(5 bits)

Ctrl 1..3 according table 2 of [@!ETSI-TETRA-ISI].

Value|Sub block 1|Sub block 2
-----|-----------|-----------
000  |normal     |normal     
001  |C stolen   |normal     
010  |U stolen   |normal     
011  |C stolen   |C stolen   
100  |C stolen   |U stolen   
101  |U stolen   |C stolen   
110  |U stolen   |U stolen   
111  |O&M ISI block          

Ctrl 4..5 according table 3 of [@!ETSI-TETRA-ISI].

Value|Sub block 1      |Sub block 2      
-----|-----------------|-----------------
  00 |BFI no errors    |BFI no errors    
  01 |BFI no errors    |BFI with error(s)
  10 |BFI with error(s)|BFI no error(s)  
  11 |BFI with error(s)|BFI with error(s)

NOTE: The meaning of C4 and C5 is outside the scope of the present document

### C bit: Failed Crypto operation indication
This bit may be set to "1" if a decryption (encrypted audio along the circuit switched mobile network, decryption at the RTP sender forwarding this audio) operation could not be performed successfully for the specific half-block. Consequently, the encryption status of the half-block audio data is unknown. Implementation of an RTP receiver has to take into account "C bit" when forwarding such TETRA audio data (either to a decoder directly or via TETRA infrastructure to a TETRA mobile unit), the contained audio might be scrambled - depending if the audio originally was generated as a plain-override half-block or as an encrypted half-block.

### FRAME_NR: FN (5 bits)
The frame number bits contain an uplink frame number as defined in table 8 of [@!ETSI-TETRA-ISI].
If no frame number is available the FRAME_NR value  **SHALL** be set to 00000.

### R: Audio Signal Relevance (3 bits)
The Audio Signal Relevance bits contain information about the Relevance of the voice packet contained here.

R 1

0: no audio signal relevance propagated (R2 and R3 do not contain any valid information)

1: audio signal relevance propagated in R2 and R3

R 2..3
According to table 1 of [@!BDBOS-BIP20]


value|relevance                                                
-----|---------------------------------------------------------
 00  |no audio signal relevance (level ? -72 dBm0)             
 01  |low audio signal relevance (-52dBm0 ? level > -72dBm0)   
 10  |medium audio signal relevance (-32dBm0 ? level > -52dBm0)
 11  |high audio signal relevance (0dBm0 ? level > -32dBm0)    

### S: Spare (7 bits)
The S bits bits are reserved for future use and set to "0" currently.

## Payload Data
Reference [@!ETSI-TETRA-ISI] contains the definition for the generation of the codec data. Data bits D1..D137 in chapter 8 correspond to the "Bit number in speech frame" row of table 4 of [@!ETSI-TETRA-ISI].

The payload itself contains TETRA ACELP coded speech information encoded according to table 4 of [@!ETSI-TETRA-Codec].


# Payload example
The following example shows how a first and a second consecutive 30 ms frame is combined into a single 60ms RTP packet. Note: This example shows the usage of OSTE mapping.

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |1|1|  CTRL   |C|0|0|0|0|0|0|0|0|D(1)                           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                           D(137)|  S          |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |0|1|  CTRL   |C|0|0|0|0|0|0|0|0|D(1)                           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                           D(137)|  S          |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Both halves of information contain exact the same CTRL bits

# Congestion Control Considerations

Tetra uses a fixed bitrate which cannot be adjusted at all. 

Since UDP does not provide congestion control, applications that use RTP over UDP **SHOULD** implement their own congestion control above the UDP layer RFC8085 [@!RFC8085] and **MAY** also implement a transport circuit breaker RFC8083 [@!RFC8083].  Work in the RMCAT working group [@RMCAT] describes the interactions and conceptual interfaces necessary between the application components that relate to congestion control, including the RTP layer, the higher-level media codec control layer, and the lower-level transport interface, as well as components dedicated to congestion control functions.

Congestion control for RTP  **SHALL** be used in accordance with RFC 3550 [@!RFC3550], and with any applicable RTP profile; e.g., RFC 3551 [@!RFC3551]. An additional requirement if best-effort service is being used is: users of this payload format  **MUST** monitor packet loss to ensure that the packet loss rate is within acceptable parameters.

# Payload Format Parameters

This RTP payload format is identified using one media subtype (audio/TETRA) which is registered in accordance with RFC 4855 [@RFC4855] and per media type registration template from RFC 6838 [@RFC6838].

## Media Type Definition {#Media-Type-Definition}

The media type for the TETRA codec is expected to be allocated from the IETF tree once this draft turns into an RFC. This media type registration covers both real-time transfer via RTP and non-real-time transfers via stored files.

Media Type name:
:   audio

Media Subtype name:
:   TETRA

Required parameters:
:   none

Optional parameters: 

These parameters apply to RTP transfer only.

maxptime:
:   The maximum amount of media which can be encapsulated in a payload packet, expressed as time in milliseconds. The time is calculated as the sum of the time that the media present in the packet represents. The time  **SHOULD** be an integer multiple of the frame size. If this parameter is not present, the sender  **MAY** encapsulate any number of speech frames into one RTP packet.

ptime:
:   see RFC 4566 [@!RFC4566].

Security considerations:
:   See Section [] (#Security) of RFC XXXX.
      [RFC Editor: Upon publication as an RFC, please replace "XXXX" with the number assigned to this document and remove this note.]

Interoperability considerations:

Published specification:

Applications that use this media type:

This media type is used in applications needing transport or storage of encoded voice. Some examples include; Voice over IP, streaming media, voice messaging, and voice recording on recording systems.

Intended usage:
:  COMMON

# Mapping to SDP
The information carried in the media type specification has a specific mapping to fields in the Session Description Protocol [@!RFC4566], which is commonly used to describe RTP sessions. When SDP is used to specify sessions employing the TETRA codec, the mapping is as follows:

Media Type name:
:   audio

Media subtype name:
:   TETRA

Required parameters:
:   none

Optional parameters:
:   none

Mapping Parameters into SDP
:   The information carried in the media type specification has a specific mapping to fields in the Session Description Protocol [@!RFC4566], which is commonly used to describe RTP sessions. When SDP is used to specify sessions employing the TETRA codec, the mapping is as follows:
  -  The media type ("audio") goes in SDP "m=" as the media name.
  -  The media subtype (payload format name) goes in SDP "a=rtpmap" as the encoding name. The RTP clock rate in "a=rtpmap" MUST be 8000.
  -  The parameters "ptime" and "maxptime" go in the SDP "a=ptime" and "a=maxptime" attributes, respectively.
  -  Any remaining parameters go in the SDP "a=fmtp" attribute by copying them directly from the media type parameter string as a semicolon-separated list of parameter=value pairs.

Here is an example SDP session of usage of TETRA:

    m=audio 49120 RTP/AVP 99
    a=rtpmap:99 TETRA/8000
    a=maxptime:60
    a=ptime:60

## Offer/Answer Considerations

The following considerations apply when using SDP Offer-Answer procedures to negotiate the use of TETRA payload in RTP:

  -  In most cases, the parameters "maxptime" and "ptime" will not affect interoperability; however, the setting of the parameters can affect the performance of the application. The SDP offer-answer handling of the "ptime" and "maxptime" parameter is described in RFC3264 [@RFC3264].
  - Integer multiples of 30ms **SHALL** be used for ptime.  It is recommended to use packet size of 60ms. There is no need that ptime and maxptime parameters are negotiated symmetrically.
  -  Any unknown parameter in an offer  **SHALL** be removed in the answer.

##  Declarative SDP Considerations

For declarative media, the "ptime" and "maxptime" parameter specify the possible variants used by the sender.

# IANA Considerations

This memo requests that IANA registers [audio/TETRA] from section [](#Media-Type-Definition). The media type is also requested to be added to the IANA registry for "RTP Payload Format MIME types" (http://www.iana.org/assignments/rtp-parameters).

# Security Considerations {#Security}

RTP packets using the payload format defined in this specification are subject to the security considerations discussed in the RTP specification [@!RFC3550] , and in any applicable RTP profile. The main security considerations for the RTP packet carrying the RTP payload format defined within this memo are confidentiality, integrity and source authenticity. Confidentiality is achieved by encryption of the RTP payload. Integrity of the RTP packets through suitable cryptographic integrity protection mechanism. Cryptographic systems may also allow the authentication of the source of the payload. A suitable security mechanism for this RTP payload format should provide confidentiality, integrity protection and at least source authentication capable of determining if an RTP packet is from a member of the RTP session or not.

Note that the appropriate mechanism to provide security to RTP and  payloads following this memo may vary. It is dependent on the application, the transport, and the signaling protocol employed.  Therefore a single mechanism is not sufficient, although if suitable  the usage of SRTP [@RFC3711] is recommended. Other mechanism that may be used are IPsec [@RFC4301] and TLS [@RFC5246] (RTP over TCP), but also other alternatives may exist.


<reference anchor='ETSI-TETRA-ISI' target='http://www.etsi.org/deliver/etsi_ts/100300_100399/1003920306/01.01.01_60/ts_1003920306v010101p.pdf'>
    <front>
        <title>TS 100 392-3-6; Terrestrial Trunked Radio (TETRA); Voice plus Data (V+D); Part 3: Interworking at the Inter-System Interface (ISI); Sub-part 6: Speech format implementation for circuit mode transmission V1.1.1</title>
        <author fullname='ETSI'>
            <organization>European Telecommunications Standards Institute</organization>
            <address>
                <email>editor@etsi.org</email>
            </address>
        </author>
        <date year='2003'/>
    </front>
</reference>

<reference anchor='ETSI-TETRA-Codec' target='http://www.etsi.org/deliver/etsi_en/300300_300399/30039502/01.03.01_60/en_30039502v010301p.pdf'> 
    <front>
        <title>EN 300 395-2; Terrestrial Trunked Radio (TETRA); Speech codec for full-rate traffic channel; Part 2: TETRA codec V1.3.1</title>
        <author fullname='ETSI'>
            <organization>European Telecommunications Standards Institute</organization>
            <address>
                <email>editor@etsi.org</email>
            </address>
        </author>
        <date year='2005'/>
    </front>
</reference>

<reference anchor='BDBOS-BIP20'> 
    <front>
        <title>BIP 20 QOS Dienstgüte-Parameter BOS-Interoperabilitätsprofil für Endgeräte zur Nutzung im Digitalfunk BOS; Version 2014-04 - Revision 2</title>
        <author fullname='BDBOS'>
            <organization>Bundesanstalt für den Digitalfunk der Behörden und Organisationen mit Sicherheitsaufgaben</organization>
            <address>
                <uri>http://www.bdbos.bund.de</uri>
            </address>
        </author>
        <date year='2014'/>
    </front>
</reference>

<reference anchor='RMCAT' target='https://datatracker.ietf.org/wg/rmcat/about/'>
    <front>
        <title>RTP Media Congestion Avoidance Techniques (rmcat) Working Grooup</title>
        <author fullname='IETF'>
            <organization>RTP Media Congestion Avoidance Techniques (rmcat) Working Group</organization>
            <address>
                <uri>https://datatracker.ietf.org/wg/rmcat/about/</uri>
            </address>
        </author>
        <date year='2018'/>
    </front>
</reference>

