# [Backstage](https://backstage.io)

## Gitlab登入

### [gitlab.com 應用程式設定](https://backstage.io/docs/auth/gitlab/provider#gitlabcom-application-setup)
1. 先到Gitlab設定一個應用程式，先點擊左上角的使用者頭像，選擇 `Preferences` > `Applications`
2. 點擊右側的 `Add New Application`
3. 名稱隨便填，Redirect URI填入 `http://localhost:7007/api/auth/gitlab/handler/frame`
4. `Scopes` 權限選擇 `read_user` & `read_repository` & `write_repository` & `openid` & `profile` & `email`
5. 點擊 `Save Application`
6. 記住 `Application ID` & `Secret`，會對應到 `app-config.yaml` 內的 `clientId` & `clientSecret`

### [app-config.yaml 設定](https://backstage.io/docs/auth/gitlab/provider/#gitlabcom-application-setup)
打開 `app-config.yaml`，找到 `auth` 下的 `providers`，新增以下內容：
```yaml
auth:
  environment: development
  providers:
    gitlab:
      development:
        clientId: {Gitlab的Application ID}
        clientSecret: {Gitlab的Secret}
        callbackUrl: {Gitlab的redirect_url forExample: http://localhost:7007/api/auth/gitlab/handler/frame}
        # audience: {自架Gitlab的host}  # 如果Gitlab是自己架設的，則需要設定此處，forExample: https://gitlab.company.com
```

### [新增後端Gitlab登入Provider](https://backstage.io/docs/auth/identity-resolver/)

打開 `packages/backend/src/plugins/auth.ts`，在 `createRouter.providerFactories`，新增以下內容：

```ts
  return await createRouter({
    // ...此處省略
    providerFactories: {
      // ...此處省略
      gitlab: providers.gitlab.create({
        signIn: {
          resolver(_, ctx) {
            const userRef = 'user:default/guest'; // Must be a full entity reference
            return ctx.issueToken({
              claims: {
                sub: userRef, // The user's own identity
                ent: [userRef], // A list of identities that the user claims ownership through
              },
            });
          },
        },
      }),
    },
  });
```
### [新增前端Gitlab登入區塊與彈窗](https://backstage.io/docs/auth/#adding-the-provider-to-the-sign-in-page)

打開 `packages/app/src/App.tsx`，在 `SignInPage` 的 `providers`，新增以下內容：

```tsx
import { gitlabAuthApiRef } from '@backstage/core-plugin-api';
import { SignInPage } from '@backstage/core-components';

// ...此處省略

const app = createApp({
  
  // ...此處省略

  components:{
    SignInPage: props => (
      <SignInPage
        {...props}
        auto
        provider={{
          id: 'gitlab-auth-provider',
          title: 'GitLab',
          message: 'Sign in using GitLab',
          apiRef: gitlabAuthApiRef,
        }}
      />
    ),
  }
});
```

[＊官方身份驗證問題排除集錦](https://backstage.io/docs/auth/troubleshooting/)


最後，執行以下的終端指令來啟動Dev Server：
```sh
yarn install
yarn dev
```