# Pseudonymized Identifier

## How to create a pseudonymized identifier
- An eliptic curve (EC) point needs to be created from the identifier following the specs of the eHealth cookbook
- The EC point needs to be blinded by moving the EC point by a random scalar
- The point needs to be pseudonymized into the domain vitalink_v1 by calling the eHealth API for pseudonymization
- The response of eHealth will be a pseudonym in transit which contains among other things x, y and transitInfo.
- The x and y coordinates are base64 encoded. A new EC point needs to be created using these coordinates.
- The new EC point can be unblinded using the previously generated random scalar.
- The eHealth response needs to be updated with the newly unblinded coordinate.
- The now unblinded eHealth response needs to be base64 encoded to use as the patient identifier in the calls to vitalink.

## How to use a pseudonymized identifier
### CREATE
- The patient resource needs an identifier with a system, value and extensions.
- The system is mandatory and needs to be https://www.ehealth.fgov.be/standards/fhir/core/NamingSystem/ssin
- The value is the base 64 pseudonym in vitalink_v1 domain
- The extension is necessary to mark the identifier as a pseudonym
- This is an example of a correct patient resource JSON:

"patient": {
		"type": "Patient",
		"identifier": {
			"system": "https://www.ehealth.fgov.be/standards/fhir/core/NamingSystem/ssin",
			"value": "{{BASE_64_VITALINK_PSEUDONYM}}",
			"_value": {
				"extension": [
					{
						"extension": [
							{
								"url": "marker",
								"valueBoolean": true
							}
						],
						"url": "https://www.ehealth.fgov.be/standards/fhir/infsec/StructureDefinition/be-ext-pseudonymization"
					}
				]
			}
		}
	},

### SEARCH
- A post request needs to be constructed with URL encoded parameters in the body
- The mandatory patient.identifier parameter needs to be present.
- The presence of a system and a value is enforced in the parameters
- A correct example of a search URL is the following:
  https://apps-acpt.vitalink-services.be/vault/api/r4/AllergyIntolerance/_search
- A correct example of the body would be:
  patient.identifier: "https://www.ehealth.fgov.be/standards/fhir/core/NamingSystem/ssin|{{BASE_64_VITALINK_PSEUDONYM}}"
  
https://apps-acpt.vitalink-services.be/vault/api/r4/AllergyIntolerance/_search?patient.identifier=https://www.ehealth.fgov.be/standards/fhir/core/NamingSystem/ssin|{{BASE_64_VITALINK_PSEUDONYM}}
