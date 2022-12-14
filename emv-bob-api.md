# EMV & BoB

This note describes modifications to adopt BoB APIs to EMV payment cards.


## Existing Objects

### Token Identifier

EMV tokens (cards) are identified by a `tokenId` defined as the "SHA-256 of ICC Public Key Certificate ([EMV tag 9F46](https://emvlab.org/emvtags/show/t9F46/))". No model updates, except for the description, for the existing `tokenId` object required.

    tokenId:
      description: |
        Token identifier:
        - MTS7: Token JWK fingerprint (per RFC 7638)
        - EMV: base64url-encoded SHA-256 of the ICC Public Key Certificate (EMV tag 9F46)
      type: string
      example: "f7350c49-1860-4674-94dd-826a3d5ac4dc"


## New Objects

### Token Type

In order to distinguish between mts7 and emv tokens, an optional and informational `tokenType` object is defined. It is defined as non-enum string as it is expected that other token types may be defined in the future.

    tokenType:
      description: |
        Type of token referred to by a tokenId. Currently defined values:
        - mts7: MTS7 Travel card application
        - emv: EMV Contactless payment card
      type: string


### EMV Transaction

An EMV transaction

    emvTransaction:
      description: Contactless payment card (EMV) transaction information
      type: object
      properties:
        transactionId:
          description: Payment transaction identifier
          type: string
          example: "4B3249E2-EFFD-4C42-B604-D632C58E46FC"
        tokenVariant:
          description: EMV card variant indicating open or closed loop
          type: string
          enum:
            - open
            - closed
          default: open


## Ticket API

All objects with `tokenId` is updated to also include (an optional) new `tokenType`. If a `tokenId` is specified without `tokenType`, MTS7 is assumed.


## Validation & Inspection API

`tokenId`, `tokenType` and `emvTransaction` is added on the top level `ticketEvent` object.
