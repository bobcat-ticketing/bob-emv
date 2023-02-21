# EMV & BoB

This note describes modifications to adopt BoB APIs to EMV payment cards.


## Existing Objects

### Token Identifier

EMV tokens (cards) are identified by a `tokenId` defined as the "SHA-256 of ICC Public Key Certificate ([EMV tag 9F46](https://emvlab.org/emvtags/show/t9F46/))". No model updates, except for the description, for the existing `tokenId` object required.

    tokenId:
      description: |
        Token identifier:
        - mts7: Token JWK fingerprint (per RFC 7638)
        - emv: base64url-encoded SHA-256 of the ICC Public Key Certificate (EMV tag 9F46)
      type: string
      example: "Rnp2RTk0elFMMjFRTl9uMWhSWmhlbGowNFR5RWRmYWJyZUY1NkZPTUxpUQ"


## New Objects

### Token Type

In order to distinguish between mts7 and emv tokens, an optional and informational `tokenType` object is defined. It is defined as non-enum string as it is expected that other token types may be defined in the future.

    tokenType:
      description: |
        Type of token referred to by a tokenId. Currently defined values:
          - mts7: MTS7 Travel card application
          - emv: EMV Contactless payment card
        Implementors are advised to accept other values for maximum backwards
        compatibility.
      type: string
      x-extensible-enum:
        - mts7
        - emv


### EMV Transaction

An EMV transaction to be included as a property of the top level `ticketEvent` object. The `aid` property can be fetched from [EMV tag 9F06](https://emvlab.org/emvtags/show/t9F06/).

    emvTransaction:
      description: Contactless payment card (EMV) transaction information
      type: object
      required:
        - tokenId
        - aid
      properties:
        tokenId:
          $ref: '#/definitions/tokenId'
        aid:
          description: Application Identifier
          type: string
          pattern: "^[0-9A-Fa-f]{10,32}$"
          example: A0000008381010
        deviceId:
          description: Device (terminal) identifier
          type: string
          example: "2EF23D68-F810-4BC2-82E6-C17795D48F60"
        deviceTransactionId:
          description: Device (terminal) transaction identifier
          type: string
          example: "7997D984-7164-4BC3-9B0F-D4F575F6780F"
        paymentTransactionId:
          description: Payment transaction identifier
          type: string
          example: "4B3249E2-EFFD-4C42-B604-D632C58E46FC"
        closed:
          description: Card is closed loop
          type: boolean


## Ticket API

All objects with `tokenId` is updated to also include (an optional) new `tokenType`. If a `tokenId` is specified without `tokenType`, MTS7 is assumed.


## Ticket, Validation & Inspection API

Properties `emvTransaction` are added on the top level `ticketEvent` object.


## Participant Metadata

For use in BoB participant metadata, the following object is defined for `participantInfo`:

    emv:
      title: EMV Participant Information
      type: object
      required:
        - aid
      properties:
        aid:
          description: Application Identifier
          type: string
          pattern: "^[0-9A-Fa-f]{10,32}$"
          example: A0000008381010
        keys:
          title: CA Public Keys
          type: array
          items:
            type: object
            properties:
              index:
                type: integer
                description: Certification Authority Public Key Index
                example: 4
              kty:
                type: string
                description: JWA key type (always "RSA" for EMV cards)
                example: "RSA"
              n:
                type: string
                description: RSA modulus parameter (required for kty=RSA)
                example: "vuYlNydToNVIeSS3lcKfqoq1bb4vkrB0a2fwqE0IyJZ..."
              e:
                type: string
                description: RSA exponent parameter (required for kty=RSA)
                example: Aw
              exp:
                type: integer
                description: Key expire timestamp (seconds since epoch)
                example: 2240521200
