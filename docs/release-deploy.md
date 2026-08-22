# Deploy por GitHub Releases

## Fluxo

A esteira usa releases como artefato imutável de implantação:

```text
feature/* → PR → dev → Release test-vX.Y.Z → workspace Test
                                      ↓
                         Release prod-vX.Y.Z → workspace Prod
```

- `dev`: origem do release de teste.
- `test`: validação do release no workspace Test.
- `main`: branch de produção; releases de produção apontam para `test` após homologação.
- A branch `prod` foi removida.

## Configuração obrigatória

Configure no GitHub, em **Settings → Environments**, os ambientes `test` e `prod`.

Para cada ambiente, crie a variável:

```text
FABRIC_WORKSPACE_ID=<ID do workspace Fabric do ambiente>
```

No nível do repositório ou dos ambientes, configure os secrets:

```text
FABRIC_TENANT_ID=523e4e0c-e4db-4b38-8d89-b2e20a984366
FABRIC_CLIENT_ID=<Application ID do service principal>
FABRIC_CLIENT_SECRET=<Secret do service principal>
```

O service principal precisa de permissão Contributor ou superior nos workspaces Test e Prod. O secret nunca deve ser commitado.

## Criar releases

Depois de aprovar e mesclar alterações em `dev`:

```powershell
gh release create test-v0.1.0 --target dev --title "Test v0.1.0" --generate-notes
```

Após homologar o workspace Test:

```powershell
gh release create prod-v0.1.0 --target test --title "Prod v0.1.0" --generate-notes
```

Também é possível executar manualmente o workflow `Fabric Release Deploy`, escolhendo `test` ou `prod` e informando a ref.

## Segurança

- Releases são imutáveis e identificam exatamente o código implantado.
- O workflow usa `environment: test` ou `environment: prod` para registrar deployments.
- `test` e `main` continuam protegidas contra push direto.
- O CI deve passar antes de promover alterações.