# NFC Data Format Specification
1. The top layer format of the NFC chip is a NDEF message.
1. The message has a **NDEF record** of mime type **application/vnd.prusa3d.mat.nfc**.
   1. The record should be the first message in the message. Adding further records is allowed.
   1. The payload of the record consists of:
      1. **Meta section** (CBOR map)
         1. Always at the very beginning of the payload.
         1. Contains information about other regions:
            1. Region is a part of the payload allocated for the respective section.
            1. A section does not have to fill the whole region.
      1. **Main section** (CBOR map)
         1. Positioned at the beginning of the main region.
         1. Intended for static information, not intended to be updated by the printers.
            1. The only situation where this region needs to be updated would be when the container is being repurposed.
      1. **Main section signature** (optional, CBOR byte string)
         1. Immediately follows the main section.
         1. Presence determined by the `signed` field in the main section.
         1. Must fit in the main region.
         1. If present, the signature **MUST** be encoded as CBOR byte string.
         1. We **heavily recommend using ecdsa secp256k1** signature encoded in the DER format (and then in CBOR).
            1. Having the same signature type everywhere enables optimizations for firmware developers.
      1. **Auxiliary section** (optional, CBOR map)
         1. Positioned at the beginning of the aux region.
         1. Intended for dynamic information, intended to be updated by the printers.

## Specification common to all sections
1. Data of all sections in the specification are represented as a CBOR map.
   1. Keys of the map are integers. Semantics of the keys are specific to each section.
   1. The reader must be able to skip all unknown keys, of any data type.
1. `enum` fields are encoded as an integer, according to the enum field mapping
1. `enum_array` fields are encoded as CBOR arrays of integers, according to the field mapping
1. `timestamp` fields are encoded as UNIX timestamp integers
1. `❗` means required, `❕` means recommended

## Meta section
1. CBOR map, keys are integers.
1. Allows defining of region offsets (within the NDEF payload) and sizes.
   1. Main region is always present. If its offset is not defined, it is assumed to start right after the meta section.
   1. Auxiliary region is optional (although heavily recommended). If the offset is not specified for it, it is considered not present.

### Fields list
{{ fields_table("meta_fields") }}

## Main section
1. CBOR map, keys are integers.
1. Contains material information that does not change during the material lifetime.
   1. This section can possibly be locked by the manufacturer.

### Fields list
{{ fields_table("main_fields") }}

#### `material_class` items
{{ enum_table("material_class_enum") }}

#### `material_type` items
{{ enum_table("material_type_enum") }}

#### `tags` items
{{ enum_table("tags_enum") }}

## Aux section

### Fields list
{{ fields_table("aux_fields") }}

## Notes, recommendations
   1. **The record shall not be split into chunks.**
   1. We recommend to **expand the payload of the NDEF record so that the whole available memory of the NFC tag is used.**
      1. The idea is that the factory can fully lock the blocks containing the NDEF  headers and would only the memory pool open. (Possibly write protecting  part of it as well).
      1. Part of the memory pool is auxiliary section, which we want to be able to write to without having to edit other structures/shuffle things around.

   1. The meta section allows the manufacturers to configure the payload structure so that it fits their purposes and the used NFC tag specifics:
      1. The auxiliary region can be adjusted or even omitted, based on the NFC tag available memory.
         1. It is not specified that the auxiliary region must be after the main region. Auxiliary region can possibly be located **before** the main region, if the meta section says so. We however recommend **putting the main region before the auxiliary region**, so that it can be protected together with the NDEF header.
      1. NFC tags can allow block-level locking (`LOCK BLOCK`, making the data completely read-only).
         1. We recommend **aligning the regions with the NFC tag blocks** (typically 4 B).
         1. The NDEF header and the meta section can possibly be fully locked after the first write.
         1. **We do not recommend locking the main region** this way. We aim to keep the NFC tags reusable for example for refills.
      1. NFC tags can also support page-level password protection (`PROTECT PAGE`).
         1. **Protecting the auxiliary region is not supported at the moment** by this specification. The auxiliary region needs to be writable without any protection at all times.
         1. In case of protecting the main region, we recommend:
            1. Use a unique password for each package instance.
            1. Put the password on a label somewhere on the container.
               1. The label should not be accessible without unpacking the container.
               1. The label should be attached on the container itself.
               1. Make the password readable for  both humans and machines (typically text + QR code).
