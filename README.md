# INACTIVE: This repository is no longer maintained

---

# aws-kms-sign-csr

## Overview

### What it is
- A Python script, `aws-kms-sign-csr.py`, that re-signs a certificate signing request (CSR) with an asymmetric key stored in AWS KMS.
- It replaces the public key in the CSR with the KMS public key and signs the CSR through the KMS `Sign` API. The private key never leaves KMS.
- It supports RSA (sha256, sha384, sha512) and ECDSA (sha224, sha256, sha384, sha512). It uses `boto3`, `pyasn1`, and `pyasn1-modules`.

### How it relates to other repos
- It is a fork of `g-a-d/aws-kms-sign-csr` on GitHub (git remote `upstream`).

### How to run or deploy
- Create a virtualenv and run `pip3 install -r requirements.txt`.
- Run `./aws-kms-sign-csr.py --region <region> --keyid <key id or alias> --hashalgo sha256 test.csr > new.csr`. Add `--signalgo ECDSA` for EC keys.
- It uses the standard AWS credentials of the caller.
- It is not maintained. Use this only as a reference.

### Current status
- Inactive and no longer used.

---

Given an existing CSR (in PEM format) and a keypair in AWS KMS, this script:
* updates the public key to the public key of the asymmetric keypair
* signs the CSR with the private key of the asymmetric keypair

## Why would I want to do this?

You may have a use-case where you're signing arbitrary data using KMS, but checking
this signature against a certificate (or, by extension, checking that the certificate
has been chained from a trusted root or intermediate).

This script allows you to generate a CSR which uses the private key in KMS, which
can then be signed by your PKI. From here you can sign your arbitrary data using
KMS and you've maintained the security of your private key, as it has never left
KMS.

Note that this does NOT sign the CSR with a CA to make it into a bona fide certificate:
a CSR is signed with the private key of the generator so that the CA can ensure
that the public key is owned by the person who is requesting the certificate, and
this script re-signs with the private key held in KMS.

## Installation

    # create a new virtualenv
    python3 -m venv aws-kms-sign-csr
    . aws-kms-sign-csr/bin/activate
    # install prerequisite modules
    pip3 install -r requirements.txt

## Usage

### RSA

    # generate a PEM csr - the key doesn't matter as it will be replaced
    openssl req -new -newkey rsa:2048 -keyout /dev/null -nodes -out test.csr
    ./aws-kms-sign-csr.py --region eu-west-1 --keyid alias/mykeyalias --hashalgo sha256 test.csr > new.csr

### ECDSA
    # Create a fake key
    openssl ecparam -genkey -name secp256k1 -out fake.key -genkey
    # Create CSR from fake key
    openssl req -new -key fake.key -out test.csr
    # Update CSR using KMS key
    ./aws-kms-sign-csr.py --region eu-west-1 --keyid alias/mykeyalias --hashalgo sha256 --signalgo ECDSA test.csr > new.csr


The script will use your existing AWS credentials: to override use environment variables per https://boto3.amazonaws.com/v1/documentation/api/latest/guide/configuration.html

The key ID can be a key ARN, an actual key ID, a key alias (prefixed with alias/), or a key alias ARN. See https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kms.html#KMS.Client.sign for more info.

## Limitations

* only supports RSA with sha256, sha384 and sha512 and ECDSA with sha224, sha256, sha384, sha512 at time of writing
* should have better error handling
* should have better handling of boto profiles
