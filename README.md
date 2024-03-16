# Sim swap defence

A collection of ideas for defending against sim-swap attacks.

## Why

Several organisations still support SMS for OTP, often making it mandatory, and
even when it isn't mandatory as the primary 2FA method it's often still enabled
as a fallback, leaving you vulnerable. An attacker will always take the easiest
path, you may have a passkey set up or even a hardware fido token but if its
possible to revert to sms-based OTP then you're just as vulnerable to a sim-swap
as you would be without these extra tokens.

## How a sim-swap attack works

A sim-swap attack occurs when an attacker manages to convince or pressure your
mobile carrier into executing a sim-swap of your number over to a sim that the
attacker controls. Once this is completed any text messages or calls sent to
your number will go to the attacker, potentially giving them access to your 2FA
codes. Typically the first sign of this happening is you lose mobile service
which could easily be initially attributed to a network fault. By the time you
realise what's happened your accounts may have already been compromised.

Mobile carriers should have reasonably decent practices to prevent sim-swaps,
such as account verification and additional checks when a sim-swap is requested.
However this isn't bulletproof and they do still occur. It's been know for
sim-swap attacks to be carried out with the help of an insider.

## Mitigating a sim-swap attack

These are some ways to mitigate a sim-swap attack.

Option 0: Don't use SMS 2FA

The first option should always be to try to avoid any form of SMS 2FA. For
clarity you should always have *some* kind of 2FA, even if SMS is the only
option, however sometimes it may be possible to remove the SMS option and only
use TOTP, FIDO keys and/or backup codes. It should be noted that a phone
backup option is just as vulnerable as an SMS option if the number is taken
over.

Option 1: Get a second sim (ideally an e-sim) to use for OTP codes.

Don't give the number to anyone except the services that you need to keep secure.
Attackers trying to compromise your accounts with a sim-swap will typically get
your contact details via some means, however using this approach means they're
overwhelmingly more likely to end up with your regular number used for calls
rather than the separate number used for OTP codes. Although with this approach
your main number could still be swapped, doing so would not gain them access to
your OTP codes.

Pros:

- Simple
- Cheap - a sim only used for receiving texts costs very little
- Reasonably effective if you keep the number secret

Cons:

- Having two active sims may cause additional battery drain
- Possible implications when roaming if the sims are on different networks
- You'll lose the ability to use a second sim for another purpose, such as
  cheaper local data service while roaming
- If the number of the OTP sim were discovered an attack could still succeed

Option 2: Use a virtual mobile number to forward OTP codes

This is in my opinion the better option however it costs money and effort to
set up. The basic premise is that you rent a "virtual" mobile number from a
telephony provider such as AQL. When messages are sent to the number they are
sent to a url endpoint on a server which you host in order to receive the
messages. You can then securely obtain them via whatever means you deem
sensible.

As virtual mobile numbers are cloud-hosted and not tied to a physical sim or
e-sim, they cannot be sim-swapped by an attacker.

An example of the simplest possible implementation would be:

- OTP SMS received by virtual number
- Request sent to url endpoint
- Server sends a push notification to the mobile device using the Pushover app
  or an equivalent, containing the sender and the content of the text message

This would mitigate a sim-swap attack but would still leave the codes exposed if
either the push notification server were compromised or the phone were stolen in
an unlocked state. Depending on the security settings of the phone it may also
be possible to read the OTP codes from the lock screen.

A more secure implementation might be:

- OTP SMS received by virtual number
- Request sent to url endpoint
- OTP code derived from the message, stored on the server and linked to a random
  id token
- Push notification sent to the device but without the OTP code in it,
  containing a url with the id token in it, eg:
  https://myauth?token=randomstring123
- When the notification is opened the url will open automatically
- The myauth website, also hosted on the server, authenticates the device with
  a passkey using webauthn, making use of the device's biometric security
- If auth is successful the OTP code is returned by the server and displayed

Pros:

- Can be made very secure such that retrieving an OTP token requires biometric
  authentication
- Effectively mitigates sim-swap attacks
- Prevents OTP access in the event that the device is stolen, even in an
  unlocked state

Cons:

- Complex to set up, requires expert knowledge of several technologies
- Costs more - need to rent both a server and a virtual mobile number
- If the server breaks OTP codes may be inaccessible
- Doesn't completely eradicate the security threat as the service provider could
  still be maliciously persuaded to change the endpoint that the messages are
  posted to
