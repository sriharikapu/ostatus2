# OStatus

A Ruby toolset for interacting with the OStatus suite of protocols:

* Subscribing to and publishing feeds via PubSubHubbub
* Interacting with feeds via Salmon

## Installation

    gem install ostatus

## Usage

When your feed updates and you need to notify subscribers:

    p = OStatus::Publication.new('http://url.to/feed', ['http://some.hub'])
    p.publish

When you want to subscribe to a feed:

    token  = 'abc123'
    secret = 'def456'

    s = OStatus::Subscription.new('http://url.to/feed', token: token, secret: secret, webhook: 'http://url.to/webhook', hub: 'http://some.hub')
    s.subscribe

Your webhook URL will receive a HTTP **GET** request that you will need to handle:

    if s.valid?(params['hub.topic'], params['hub.verify_token'])
      # echo back params['hub.challenge']
    else
      # return 404
    end

Once the subscription is established, your webhook URL will be receiving HTTP **POST** requests. Among the headers of such a request will be the hub's signature on the content: `X-Hub-Signature`. You can verify the integrity of the request:

    body      = request.body.read
    signature = request.env['HTTP_X_HUB_SIGNATURE']

    if s.verify(body, signature)
      # Do something with the data!
    end

When you want to notify a remote resource about an interaction (like a comment):

    your_rsa_keypair = OpenSSL::PKey::RSA.new 2048

    salmon   = OStatus::Salmon.new
    envelope = salmon.pack(comment, your_rsa_keypair)

    salmon.post('http://remote.salmon/endpoint', envelope)

When you receive a Salmon notification about a remote interaction:

    salmon  = OStatus::Salmon.new
    comment = salmon.unpack(envelope, your_rsa_keypair)
