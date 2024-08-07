C 180
# Способы инжектирования сервиса
- @Injectable({provideIn: 'root'})
- В main.ts bootstrapApplication(AppComponent, providers: [TaskService]})...
- В компоненте @Component({..., providers: [TaskService]}), но в таком случае будут создаваться разные экземпляры сервисов для каждого компонента

### Инжектирование сервиса в сервис
- private taskService = inject(TaskService)

### Инжектирование через токен
```
В main.ts:
const TaskServiceToken = new InjectionToken<TaskService>('tasks-service-token')
bootstrapApplication(AppComponent, providers: [{provide: TaskServiceToken, useClass: TaskService}]})
В компоненте:
private taskService = inject(TaskServiceToken)
либо
constructor(@Inject(TaskServiceToken) private taskService: TaskService)
```

```
Alpha.ts:
export const ALPHA = new InjectionToken<Alpha>('alpha');
export const Alpha: NewType = [...]

В компоненте:
@Component({
providers: [{
  provide: ALPHA,
  useValue: Alpha
}]
})
...
a = inject(ALPHA);
```

# Избегание перезапусков компиляции при внесении изменений в код
```
private zone = inject(NgZone);
...
this.zone.runOutsideAngular(() => {...});
```

```
@Component({
  ...
  changeDetection: ChangeDetectionStrategy.OnPush
})
```

# Подписки
```
message$ = new BehaviorSubject<>();
...
this.message$.next(...);
...
this.messageService.message$.subscribe((message) => { // Подписка на message$
  this.cdRef.markForCheck();
})
```

Внутри html  с помощью async pipe
```
<div *ngFor="let message of messages$ | async"></div>
```

# Signals vs Observables
- Signals - для управления состоянием приложения
- Observables - для управления событиями и потоками данных
- У сигналов есть начальное значение, у обзёрваблов может не быть

### Observable to Signal
```
obs$ = toObservable(sig);
...
this.obs$.subscribe();
...
this.sig.update(...);
```
### Signal to Observable
```
newSig = toSignal(this.obs$, {initialValue: 0});
```
### Создание Observable
```
customInterval$ = new Observable((subscriber) => {
  let times = 0;
  const interval = setInterval(() => {
    if (times > 3) {
      clearInterval(interval);
      subscriber.complete();
      return;
    }
    subscriber.next({message: 'New value'});
    times++;
  }, 2000);
});
...
this.customInterval$.subscribe({
  next: (val) => console.log(val),
  complete: () => console.log('Completed'),
  error: (err) => console.log(err)
});
```

