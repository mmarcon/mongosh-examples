require('dotenv').config();

// Wrap everything into a function so we don't
// affect the shell's scope
(function seedDBWithTestData() {
    const faker = require('faker');
    // Config DB and Collection names
    const dbName = 'herzog_inc_insurance';
    const collectionName = 'customers';

    // Configure KMS Provider
    // Using GCP
    const gcpKmsProvider = {
        gcp: {
            email: process.env.gcp_email,
            privateKey: process.env.gcp_private_key,
        }
    };
    // Get a new Mongo object with the KMS provider configuration
    const keyMongo = Mongo(process.env.mongodb_connection_string, {
        keyVaultNamespace: 'encryption.__keyVault',
        kmsProvider: gcpKmsProvider
    });

    // Get a reference to the Key Vault
    const keyVault = keyMongo.getKeyVault();


    // Generate a key
    // In this example, we will use the same key for all the test documents.
    // However, this is a good use case for assigning one key per customer.
    const keyId = keyVault.createKey('gcp', {
        projectId: process.env.gcp_project_id,
        location: 'global',
        keyRing: process.env.gcp_keyring,
        keyName: process.env.gcp_key_name
    });

    // Schema map for auto encryption
    // See https://docs.mongodb.com/manual/core/security-automatic-client-side-encryption/ for
    // the details on how automatic encryption works.
    // The automatic feature of field level encryption is only available in
    // MongoDB 4.2+ Enterprise and MongoDB Atlas 4.2+ clusters.
    const schemaMap = {};
    schemaMap[`${dbName}.${collectionName}`] = {
        bsonType: 'object',
        properties: {
            birthDate: {
                encrypt: {
                    keyId: [keyId],
                    bsonType: 'string',
                    algorithm: 'AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic'
                }
            },
            email: {
                encrypt: {
                    keyId: [keyId],
                    bsonType: 'string',
                    algorithm: 'AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic'
                }
            },
            phone: {
                encrypt: {
                    keyId: [keyId],
                    bsonType: 'string',
                    algorithm: 'AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic'
                }
            },
            bankAccount: {
                encrypt: {
                    keyId: [keyId],
                    bsonType: 'object',
                    algorithm: 'AEAD_AES_256_CBC_HMAC_SHA_512-Random'
                }
            }
        }
    };
    // Create a new instance of Mongo connected to the same
    // cluster but this time with the schemaMap configured as well.
    const autoMongo = Mongo(process.env.mongodb_connection_string, {
        keyVaultNamespace: 'encryption.__keyVault',
        kmsProvider: gcpKmsProvider,
        schemaMap
    });


    // Generate a bunch of fake customer data...
    const autoMongoDb = autoMongo.getDB(dbName);
    // ... but first clean up old test data ...
    autoMongoDb.getCollection(collectionName).drop();
    const customers = [];
    for (let i = 0; i < 10000; i++) {
        const birthDate = faker.date.between(new Date('1940-01-01'), new Date('2018-12-31'));
        const state = faker.address.stateAbbr();
        const customer = {
            name: {
                first: faker.name.firstName(),
                last: faker.name.lastName()
            },
            email: faker.internet.email(),
            phone: faker.phone.phoneNumber(),
            company: {
                name: faker.company.companyName()
            },
            address: {
                zip: faker.address.zipCode(),
                street: faker.address.streetAddress(),
                state
            },
            birthDate: birthDate.toISOString().slice(0, 10),
            bankAccount: {
                name: faker.finance.accountName(),
                account: faker.finance.account(),
                bic: faker.finance.bic()
            },
            birthYear: birthDate.getFullYear()
        };
        customers.push(customer);
    }
    // ... finally, store the new data into the destination collection.
    // Fields that the schema map indicates they should be encrypted
    // will be encrypted client-side and then stored encrypted in MongoDB
    autoMongoDb.getCollection(collectionName).insertMany(customers);
})();