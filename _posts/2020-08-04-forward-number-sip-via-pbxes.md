---
layout: post
title: "Forwarding a (landline) phone number to an international (cell) phone number via an SIP account"
date: 2020-08-04
categories: VOIP SIP betamax pbx
comments: true
---

Let us first fix some terminology:

- **VOIP** (*Voice over Internet Protocol*) stands for Internet Telephony.
- The leading protocol to set up (to initiate, alter and terminate) a VOIP phone call is the *Session Initiation Protocol* (**SIP**).
- A phone number provided by a SIP provider is a **DID** number, a *Direct Inward Dialing* number.
- a private branch exchange (**PBX**) distributes calls between many inside lines (**extensions**) to a single outside line (**trunk**).
    While a PBX used to be hardware installed at the phone company that provided the outside lines, now it is usually software installed at the client who manages the inside lines.

# Forwarding Inward Calls

To forward

1. your landline number from a (solid but rather expensive) [`DID` number VOIP provider](http://backsla.sh/did)
1. to your (cell)phone number by another (shadier but rather cheap) [affordable VOIP provider](http://backsla.sh/betamax)
1. via a virtual phone system, private branch exchange (`PBX`), such as [PBXes.org](https://www.pbxes.com/)

follow these instructions:

1. Obtain a SIP account reachable by a phone number by opening an account at a DID number SIP provider, say `Sipgate`.
1. To forward  calls from this DID provider `Sipgate` to another SIP provider (without DID number), say `Intervoip`, via your virtual PBX,

    1. Open an account at `Intervoip`.
    1. Open an account at [PBXes.org](https://www.pbxes.com/).

    1. Login to [PBXes.org](https://www.pbxes.com/) and add an incoming (and outgoing) SIP trunk for `Sipgate` (labeled, say `Sipgate`) by entering your username (which is your client number), password (for your SIP account which may differ from that of your client account) and the SIP server address, here `sipgate.de`.
        Check at your DID provider `Sipgate` whether the trunk appears in the list of registered devices in the entry [Routing/Phones](https://app.sipgate.com/w0/devices).
    1. Create a global incoming rule that forwards all calls (all day long by enforcing work hours)
    
        - from your account at the DID provider `Sipgate`
        - to your account `john` at the other SIP provider `intervoip.com` by entering its SIP URI consisting of the account name and SIP server domain, here `john@sip.intervoip.com`.

1. To let `Intervoip` (or any other [`Betamax` reseller](http://backsla.sh/did)) forward the calls from your SIP account to your (cell)phone number:

    1. Download the application running on Microsoft Windows.
        Since some time, all resellers offer the MobileVoIP app of the Microsoft store.
        However, still working and available for direct download from other sources are the former branded installers, such as that of [VoipBuster](https://www.chip.de/downloads/VoipBuster_16447285.html).
    1. Once the application installed, started and you are logged on to your SIP account, open the menu entry `Options` and then in the appearing dialog the list entry `Call Settings`.
        In the entry `Call forwarding` choose `always` and add your cellphone number (including its international prefix and area code).

# Placing Outward Calls

To call others, you can do so by a regular phone call:

1. Call a local access number, of which there are a couple of them in each country, all consultable at the [geo](https://www.intervoip.com/geo) page of your [`Betamax` reseller](http://backsla.sh/did) (and with varying numbers among them), and
1. Dial on your (virtual) phone pad the destination number.

The number is called after the charged tariff has been announced.

Or you can make calls on a smartphone VOIP client:

On Android, open source clients are

- [SIPdroid](http://sipdroid.org/), from the operator of `pbxes.org`
- `Csipsimple`, which is no longer maintained

and various solid commercial ones, such as

- [Acrobits]( https://www.acrobits.net)
- [Zoiper](https://www.zoiper.com/en) which was found by me, many years ago, to be the most stable among those I test drove.

Once installed, set up your account in your VOIP client as indicated on the [SIP page](https://www.intervoip.com/sip).

More flexible and (cellphone) battery friendly is to:

1. Add to your virtual PBX at `pbxes.org`

    - an extension, say `100`
    - add an outgoing rule that forwards the calls to your `Betamax` reseller, say `Intervoip` by  entering your username, password and the SIP server address, say `sip.intervoip.com`.
        The rule can, for example, pick the account of that Betamax reseller with the cheapest tariffs to the destination number according to its international prefix (and for national landline calls for chivalry use the DID provider).

1. In your VOIP client, set up your SIP account (using TCP to establish a connection instead of UDP, thus saving battery of your cellphone when the SIP client is idly waiting for incoming calls) at `pbxes.org` by entering the username `john-100` (where the first part before the dash `john` is the pbxes account name and the last part `100` is the extension number), password and the SIP server address `pbxes.com`

<!-- private branch exchange (PBX) = -->
<!--  -->
<!-- A telephone exchange serving a single organization, having a switchboard and associated equipment, usually located on the customer's premises; -->
<!-- provides for switching calls between any two extensions served by the exchange or between any extension and the national telephone system via a trunk to a central office. -->
<!--  -->
<!-- A switching system located on a customer’s premises that consolidates the number of inside lines (extensions) into a smaller number of outside lines (trunks). Many PBXs also provide advanced voice and data communication features. -->

<!-- The Internet Protocol Private Branch eXchange IP PBX) = -->
<!-- is telephone switching equipment that resides in a private business instead of the telephone company. -->
<!-- An IP PBX delivers employees dial-tone, the ability to conference, transfer, and dial other employees by extension number as well as many other features. -->

<!-- Direct Inward Dialing (DID) = -->
<!-- the ability to make an external telephone call to an internal extension within an organization = -->
<!-- The capability for dialing individual telephone extensions in a large organization directly from outside, without going through a central switchboard = -->
<!-- feature offered by telephone companies for use with their customers' PBX system, whereby the telephone company (telco) allocates a range of numbers all connected to their customer's PBX. -->
<!-- As calls are presented to the PBX, the number that the caller dialled is also given, so that the PBX can decide which person in the office to route the call to. -->

<!-- Session Initiation Protocol Trunking (SIP Trunking) = -->
<!-- the use of VoIP to facilitate the connection of typically a PBX to the Internet, where the Internet replaces the conventional telephone trunk, allowing a business to communicate with traditional PSTN telephone subscribers worldwide by connecting to an ITSP (Internet Telephony Service Provider). -->
<!-- a SIP Trunk is a single voice connection (call) placed over your Internet connection. -->
<!-- This VoIP "trunk" (or phone line) connects to a provider who routes your calls through their gateway. -->
<!--  -->
<!-- The phone call either originates as an IP call or is converted to one before it leaves the office. -->
<!-- This is done by the IP-PBX or a gateway device. -->
<!-- The call is then carried as data (RTP Real Time Protocol) the majority of the way over the Internet to the Internet Telephone service provider and on to a gateway where it converts back to a traditional call to PSTN for the last leg on the route. -->
<!-- Since a sizable portion of the call travels over the Internet or IP network, at almost no additional cost, the service provider can afford to offer calling plans at significantly reduced prices. -->
