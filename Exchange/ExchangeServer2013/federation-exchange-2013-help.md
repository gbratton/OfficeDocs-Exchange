---
title: 'Federation: Exchange 2013 Help'
TOCTitle: Federation
ms:assetid: 0046e2eb-6940-4941-bd5b-cbe6bffa3b94
ms:mtpsurl: https://technet.microsoft.com/library/Dd335047(v=EXCHG.150)
ms:contentKeyID: 48384773
ms.reviewer: 
manager: serdars
ms.author: serdars
author: msdmaguire
ms.topic: article
description: Federation in Exchange Server supports collaboration
f1.keywords:
- NOCSH
mtps_version: v=EXCHG.150
---

# Federation

_**Applies to:** Exchange Server 2013_

Information workers frequently need to collaborate with external recipients, vendors, partners, and customers and share their free/busy (also known as calendar availability) information. Federation in Microsoft Exchange Server 2013 helps with these collaboration efforts. *Federation* refers to the underlying trust infrastructure that supports *federated sharing*, an easy method for users to share calendar information with recipients in other external federated organizations. To learn more about federated sharing, see [Sharing](sharing-exchange-2013-help.md).

> [!IMPORTANT]
> This feature of Exchange Server 2013 isn't fully compatible with Office 365 operated by 21Vianet in China and some feature limitations may apply. For more information, see [Office 365 Operated by 21Vianet](/office365/servicedescriptions/office-365-platform-service-description/office-365-operated-by-21vianet).

## Key terminology

The following list defines the core components associated with federation in Exchange 2013.

- **application identifier (AppID)**: A unique number generated by the Azure Active Directory authentication system to identify Exchange organizations. The AppID is automatically generated when you create a federation trust with the Azure Active Directory authentication system.

- **delegation token**: A Security Assertion Markup Language (SAML) token issued by the Azure Active Directory authentication system that allows users from one federated organization to be trusted by another federated organization. A delegation token contains the user's email address, an immutable identifier, and information associated with the offer for which the token is issued for action.

- **external federated organization**: An external Exchange organization that's established a federation trust with the Azure Active Directory authentication system.

- **federated sharing**: A group of Exchange features that leverage a federation trust with the Azure Active Directory authentication system to work across Exchange organizations, including cross-premises Exchange deployments. Together, these features are used to make authenticated requests between servers on behalf of users across multiple Exchange organizations.

- **federated domain**: An accepted authoritative domain that's added to the organization identifier (OrgID) for an Exchange organization.

- **domain proof encryption string**: A cryptographically secure string used by an Exchange organization to provide proof that the organization owns the domain used with the Azure Active Directory authentication system. The string is generated automatically when using the **Enable federation trust** wizard or can be generated by using the **Get-FederatedDomainProof** cmdlet.

- **federated sharing policy**: An organization-level policy that enables and controls user-established, person-to-person sharing of calendar information.

- **federation**: A trust-based agreement between two Exchange organizations to achieve a common purpose. With federation, both organizations want authentication assertions from one organization to be recognized by the other.

- **federation trust**: A relationship with the Azure Active Directory authentication system that defines the following components for your Exchange organization:

  - Account namespace

  - Application identifier (AppID)

  - Organization identifier (OrgID)

  - Federated domains

  To configure federated sharing with other federated Exchange organizations, a federation trust must be established with the Azure Active Directory authentication system.

- **non-federated organization**: Organizations that don't have a federation trust established with the Azure Active Directory authentication system.

- **organization identifier (OrgID)**: Defines which of the authoritative accepted domains configured in an organization are enabled for federation. Only recipients that have e-mail addresses with federated domains configured in the OrgID are recognized by the Azure Active Directory authentication system and are able to use federated sharing features. The OrgID is a combination of a pre-defined string and the first accepted domain selected for federation in the **Enable federation trust** wizard. For example, if you specify the federated domain contoso.com as your organization's primary SMTP domain, the account namespace FYDIBOHF25SPDLT.contoso.com will be automatically created as the OrgID for the federation trust.

- **organization relationship**: A one-to-one relationship between two federated Exchange organizations that allows recipients to share free/busy (calendar availability) information. An organization relationship requires a federation trust with the Azure Active Directory authentication system and replaces the need to use Active Directory forest or domain trusts between Exchange organizations.

- **Azure Active Directory authentication system**: A free, cloud-based identity service that acts as the trust broker between federated Microsoft Exchange organizations. It's responsible for issuing delegation tokens to Exchange recipients when they request information from recipients in other federated Exchange organizations. To learn more, see [Azure Active Directory](https://azure.microsoft.com/services/active-directory/).

## Azure AD authentication system

The Azure Active Directory authentication system, a free cloud-based service offered by Microsoft, acts as the trust broker between your on-premises Exchange 2013 organization and other federated Exchange 2010 and Exchange 2013 organizations. If you want to configure federation in your Exchange organization, you must establish a one-time federation trust with the Azure Active Directory authentication system, so that it can become a federation partner with your organization. With this trust in place, users authenticated by Active Directory (known as *identity providers*) are issued Security Assertion Markup Language (SAML) delegation tokens by the Azure AD authentication system. These delegation tokens allow users from one federated Exchange organization to be trusted by another federated Exchange organization. With the Azure AD authentication system acting as the trust broker, organizations aren't required to establish multiple individual trust relationships with other organizations, and users can access external resources using a single sign-on (SSO) experience. For more information, see [Azure Active Directory](https://azure.microsoft.com/services/active-directory/).

## Federation trust

To use Exchange 2013 federated sharing features, you must establish a federation trust between your Exchange 2013 organization and the Azure AD authentication system. Establishing a federation trust with the Azure AD authentication system exchanges your organization's digital security certificate with the Azure AD authentication system and retrieves the Azure AD authentication system certificate and federation metadata. You can establish a federation trust by using the **Enable federation trust** wizard in the Exchange admin center (EAC) or the **New-FederationTrust** cmdlet in the Exchange Management Shell. A self-signed certificate is automatically created by the **Enable federation trust** wizard and is used for signing and encrypting delegation tokens from the Azure AD authentication system that allow users to be trusted by external federated organizations. For details about certificate requirements, see Certificate Requirements for Federation later in this topic.

When you create a federation trust with the Azure AD authentication system, an *application identifier* (AppID) is automatically generated for your Exchange organization and provided in the output of the **Get-FederationTrust** cmdlet. The AppID is used by the Azure AD authentication system to uniquely identify your Exchange organization. It's also used by the Exchange organization to provide proof that your organization owns the domain for use with the Azure AD authentication system. This is done by creating a text (TXT) record in the public Domain Name System (DNS) zone for each federated domain.

## Federated organization identifier

The *federated organization identifier* (OrgID) defines which of the authoritative accepted domains configured in your organization are enabled for federation. Only recipients that have e-mail addresses with accepted domains configured in the OrgID are recognized by the Azure AD authentication system and are able to use federated sharing features. When you create a new federation trust, an OrgID is automatically created with the Azure AD authentication system. This OrgID is a combination of a pre-defined string and the accepted domain selected as the primary shared domain in the wizard. For example, in the Edit Sharing-Enabled Domains wizard, if you specify the federated domain **contoso.com** as the primary shared domain in your organization, the **FYDIBOHF25SPDLT.contoso.com** account namespace will be automatically created as the OrgID for the federation trust for your Exchange organization.

Although typically the primary SMTP domain for the Exchange organization, this domain doesn't have to be an accepted domain in your Exchange organization and doesn't require a domain name system (DNS) proof of ownership TXT record. The only requirement is that accepted domains selected to be federated are limited to a maximum of 32 characters. The only purpose of this subdomain is to serve as the federated namespace for the Azure AD authentication system to maintain unique identifiers for recipients that request SAML delegation tokens. For more information about SAML tokens, see [SAML Tokens and Claims](/dotnet/framework/wcf/feature-details/saml-tokens-and-claims)

You can add or remove accepted domains from the federation trust at any time. If you want to enable or disable all federation sharing features in your organization, all you have to do is enable or disable the OrgID for the federation trust.

> [!IMPORTANT]
> If you change the OrgID, accepted domains, or the AppID used for the federation trust, all federation sharing features are affected in your organization. This also affects any external federated Exchange organizations, including Exchange Online and hybrid deployment configurations. We recommend that you notify all external federated partners of any changes to these federation trust configuration settings.

## Federation example

Two Exchange organizations, Contoso, Ltd. and Fabrikam, Inc., want their users to be able to share calendar free/busy information with each other. Each organization creates a federation trust with the Azure AD authentication system and configures its account namespace to include the domain used for its user's e-mail address domain.

Contoso employees use one of the following e-mail address domains: contoso.com, contoso.co.uk, or contoso.ca. Fabrikam employees use one of the following e-mail address domains: fabrikam.com, fabrikam.org, or fabrikam.net. Both organizations make sure that all accepted e-mail domains are included in the account namespace for their federation trust with the Azure AD authentication system. Rather than requiring a complex Active Directory forest or domain trust configuration between the two organizations, both organizations configure an organization relationship with each other to enable calendar free/busy sharing.

The following figure illustrates the federation configuration between Contoso, Ltd. and Fabrikam, Inc.

**Federated sharing example**

![Federation Trusts and Federated Sharing.](images/Dd335047.310f0698-b67d-4b0e-91e4-231c6e9db952(EXCHG.150).gif "Federation Trusts and Federated Sharing")

## Certificate requirements for Federation

To establish a federation trust with the Azure AD authentication system, either a self-signed certificate or an X.509 certificate signed by a certification authority (CA) must be created and installed on the Exchange 2013 server used to create the trust. We strongly recommend using a self-signed certificate, which is automatically created and installed using the **Enable federation trust** wizard in the EAC. This certificate is used only to sign and encrypt delegation tokens used for federated sharing and only one certificate is required for the federation trust. Exchange 2013 automatically distributes the certificate to all other Exchange 2013 servers in the organization.

If you want to use an X.509 certificate signed by an external CA, the certificate must meet the following requirements:

- **Trusted CA**: If possible, the X.509 Secure Sockets Layer (SSL) certificate should be issued from a CA trusted by Windows Live. However, you can use certificates issued by CAs that aren't currently certified by Microsoft. For a current list of trusted CAs, see [Trusted root certification authorities for federation trusts](trusted-root-certification-authorities-for-federation-trusts-exchange-2013-help.md).

- **Subject key identifier**: The certificate must have a subject key identifier field. Most X.509 certificates issued by commercial CAs have this identifier.

- **CryptoAPI cryptographic service provider (CSP)**: The certificate must use a CryptoAPI CSP. Certificates that use Cryptography API: Next Generation (CNG) providers aren't supported for federation. If you use Exchange to create a certificate request, a CryptoAPI provider is used. For more information, see [Cryptography API: Next Generation](/windows/win32/seccng/cng-portal).

- **RSA signature algorithm**: The certificate must use RSA as the signature algorithm.

- **Exportable private key**: The private key used to generate the certificate must be exportable. You can specify that the private key be exportable when you create the certificate request using the **New Exchange certificate** wizard in the EAC or the [New-ExchangeCertificate](/powershell/module/exchange/New-ExchangeCertificate) cmdlet in the Shell.

- **Current certificate**: The certificate must be current. You can't use an expired or revoked certificate to create a federation trust.

- **Enhanced key usage**: The certificate must include the enhanced key usage (EKU) type **Client Authentication (1.3.6.1.5.5.7.3.2)**. This usage type is used to prove your identity to a remote computer. If you use the EAC or the Shell to generate a certificate request, this usage type is included by default.

> [!NOTE]
> Because the certificate isn't used for authentication, it doesn't have any subject name or subject alternative name requirements. You can use a certificate with a subject name that's the same as the host name, the domain name, or any other name.

## Transitioning to a new certificate

The certificate used to create the federation trust is designated as the current certificate. However, you may need to install and use a new certificate for the federation trust periodically. For example, you may need to use a new certificate if the current certificate expires or to meet a new business or security requirement. To ensure a seamless transition to a new certificate, you must install the new certificate on your Exchange 2013 server and configure the federation trust to designate it as the new certificate. Exchange 2013 automatically distributes the new certificate to all other Exchange 2013 servers in the organization. Depending on your Active Directory topology, distribution of the certificate may take a while. You can verify the certificate status using the [Test-FederationTrustCertificate](/powershell/module/exchange/Test-FederationTrustCertificate) cmdlet in the Shell.

After you verify the certificate's distribution status, you can configure the trust to use the new certificate. After switching certificates, the current certificate is designated as the previous certificate, and the new certificate is designated as the current certificate. The new certificate is published to the Azure AD authentication system, and all new tokens exchanged with the Azure AD authentication system are encrypted using the new certificate.

> [!NOTE]
> This certificate transition process is used only by federation. If you use the same certificate for other Exchange 2013 features that require certificates, you must take the feature requirements into consideration when planning to procure, install, or transition to a new certificate.

## Firewall Considerations for Federation

Federation features require that the Mailbox and Client Access servers in your organization have outbound access to the Internet by using HTTPS. You must allow outbound HTTPS access (port 443 for TCP) from all Exchange 2013 Mailbox and Client Access servers in the organization.

For an external organization to access your organization's free/busy information, you must publish one Client Access server to the Internet. This requires inbound HTTPS access from the Internet to the Client Access server. Client Access servers in Active Directory sites that don't have a Client Access server published to the Internet can use Client Access servers in other Active Directory sites that are accessible from the Internet. The Client Access servers that aren't published to the Internet must have the external URL of the Web services virtual directory set with the URL that's visible to external organizations.