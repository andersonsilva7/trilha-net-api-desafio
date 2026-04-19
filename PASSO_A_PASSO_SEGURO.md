# Passo a passo de setup seguro (sem credenciais no Git)

Use este guia em qualquer máquina nova para configurar e subir a API `trilha-net-api-desafio` sem expor segredos.

## 1) Pré-requisitos

- .NET SDK 8 instalado
- Git
- Acesso ao SQL Server do Azure (ou outra base SQL Server)

## 2) Clonar e entrar no projeto

```powershell
git clone <URL_DO_SEU_REPOSITORIO>
cd trilha-net-api-desafio
```

## 3) Restaurar dependências e validar build

```powershell
dotnet restore
dotnet build
```

## 4) Configurar conexão local com User Secrets

> O `appsettings.json` já está com placeholders (`SEU_*`), portanto não há credenciais reais no projeto.

No terminal, configure a connection string real **somente nesta máquina**:

```powershell
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:ConexaoPadrao" "Server=tcp:<SEU_SERVIDOR>,1433;Initial Catalog=Organizador;Persist Security Info=False;User ID=<SEU_USUARIO>;Password=<SUA_SENHA>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
```

## 5) Validar migrations (pós-credencial)

```powershell
dotnet tool install --global dotnet-ef
cd ./
dotnet ef migrations add InitialCreate
dotnet ef database update
```

## 6) Rodar localmente

```powershell
dotnet run
```

Abrir no navegador: `https://localhost:<porta>/swagger`

## 7) Publicar no Azure (sem segredos no código)

### Azure SQL

- Reaproveitar o mesmo SQL Server lógico.
- Criar nova database: `Organizador`.
- Definir login/senha no servidor apenas no Azure.

### App Service

- Criar um novo App Service para essa API (ex.: `aulaorganizadorapi`).
- Runtime: `.NET 8 (LTS)`.

### Configurar variável de ambiente do App Service

Adicionar em **Configuration > Application settings**:

- `ConnectionStrings__ConexaoPadrao`
- Valor da conexão real da Azure SQL (string completa)

## 8) Após finalizar os testes nesta máquina, remover o passo a passo

Se o arquivo for usado **somente local** e não for subir para o repositório:

```powershell
Remove-Item .\\PASSO_A_PASSO_SEGURO.md -Force
```

Se o arquivo já tiver sido commitado e você quiser removê-lo do repositório:

```powershell
git rm --cached .\\PASSO_A_PASSO_SEGURO.md
git commit -m "remove: temporary setup guide file"
```

---

> Dica: não grave credenciais reais em `README.md`, `appsettings*.json` ou qualquer arquivo versionado.
