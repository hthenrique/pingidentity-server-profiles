# Perfil Customizado do Ping Directory - Separação de Usuários

Este perfil customizado do Ping Directory foi configurado para separar os **usuários administrativos/sistema** dos **usuários comuns (regulares)** dentro do LDAP, organizando-os em Unidades Organizacionais (OUs) distintas sob a base DN configurada (`${USER_BASE_DN}`).

## Estrutura do DIT (Directory Information Tree)

Os usuários estão divididos nas seguintes OUs principais:

1. **Usuários Comuns (Regulares):**
   * **DN:** `ou=People,${USER_BASE_DN}`
   * **Objetivo:** Armazenar contas de usuários normais ou de clientes que acessam os serviços.
   * **Exemplos de Usuários Criados:** `user.0`, `user.1`, `user.2`, `user.3`, `user.4`.

2. **Usuários Administrativos e de Integração:**
   * **DN:** `ou=Admins,${USER_BASE_DN}`
   * **Objetivo:** Armazenar contas administrativas e contas de serviço usadas para integração entre os componentes do Ping Identity (como PingFederate e PingAccess).
   * **Exemplos de Usuários Criados:**
     * `uid=administrator,ou=Admins,${USER_BASE_DN}` (Administrador Geral)
     * `uid=access2federate,ou=Admins,${USER_BASE_DN}` (Integração do PingAccess)
     * `uid=auditor,ou=Admins,${USER_BASE_DN}` (Auditor do sistema)

3. **Grupos:**
   * **DN:** `ou=groups,${USER_BASE_DN}`
   * Contém os mapeamentos de grupos atualizados apontando para os DNs corretos sob `ou=Admins` (por exemplo, `PFAdminGroup`, `PFCryptoGroup`, `PFAuditorGroup`).

---

## Como Utilizar

Você pode aplicar este perfil no container Docker do Ping Directory de duas maneiras:

### Opção 1: Usando como Volume Local (Recomendado para Testes Locais)

No seu arquivo `docker-compose.yaml`, monte a pasta deste perfil dentro do diretório `/opt/in` do container e comente a variável `SERVER_PROFILE_URL`:

```yaml
  pingdirectory:
    image: pingidentity/pingdirectory:${PING_IDENTITY_DEVOPS_TAG}
    environment:
      # - SERVER_PROFILE_URL=https://github.com/pingidentity/pingidentity-server-profiles.git
      # - SERVER_PROFILE_PATH=baseline/pingdirectory
      - USER_BASE_DN=dc=example,dc=com
    volumes:
      - ./custom/pingdirectory/pd.profile:/opt/in
    ports:
      - 1636:1636
      - 1443:1443
```

### Opção 2: Como Perfil com Herança (Camada Adicional/Layered Profile)

Se você preferir herdar as configurações base (do profile `baseline/pingdirectory` por exemplo) e aplicar esta separação como uma camada adicional:

```yaml
  pingdirectory:
    image: pingidentity/pingdirectory:${PING_IDENTITY_DEVOPS_TAG}
    environment:
      - SERVER_PROFILE_URL=https://github.com/pingidentity/pingidentity-server-profiles.git
      - SERVER_PROFILE_PATH=custom/pingdirectory
      - SERVER_PROFILE_PARENT=BASELINE
      
      - SERVER_PROFILE_BASELINE_URL=https://github.com/pingidentity/pingidentity-server-profiles.git
      - SERVER_PROFILE_BASELINE_PATH=baseline/pingdirectory
```

---

## Arquivos Criados neste Perfil

* `pd.profile/ldif/userRoot/00-structure.ldif`: Cria as OUs `ou=People` e `ou=Admins` com ACIs ajustadas para dar permissões administrativas ao usuário em sua nova OU.
* `pd.profile/ldif/userRoot/10-users.ldif`: Popula os usuários de exemplo nos diretórios correspondentes.
* `pd.profile/ldif/userRoot/20-groups.ldif`: Define grupos administrativos mapeando os membros corretos em `ou=Admins`.
* `pd.profile/dsconfig/51-custom-delegated-admin.dsconfig`: Ajusta a configuração de direitos do Delegated Admin para mapear o DN do usuário administrativo em `ou=Admins`.
