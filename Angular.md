C 180
**Способы инжектирования сервиса**
- @Injectable({provideIn: 'root'})
- В main.ts bootstrapApplication(AppComponent, providers: [TaskService]})...
- В компоненте @Component({..., providers: [TaskService]}), но в таком случае будут создаваться разные экземпляры сервисов для каждого компонента
- 
