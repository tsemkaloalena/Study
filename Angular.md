C 180
# Сигналы


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

# HttpCLient
### providers
```
bootstrapApplication(AppComponent, {
  providers: [provideHttpClient()]
}).catch((err) => console.log(err));

либо

@NgModule({
  declarations: [
    AppComponent,
    PlacesComponent,
    // ... etc
  ],
  imports: [BrowserModule, FormsModule],
  providers: [provideHttpClient()],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

### http interceptor
Besides defining HTTP interceptors as functions (which is the modern, recommended way of doing it), you can also define HTTP interceptors via classes.

For example, the loggingInterceptor from the previous lecture could be defined like this (when using this class-based approach):
```
import {
  HttpEvent,
  HttpHandler,
  HttpInterceptor,
  HttpRequest,
} from '@angular/common/http';
import { Observable } from 'rxjs';
 
@Injectable()
class LoggingInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<unknown>, handler: HttpHandler): Observable<HttpEvent<any>> {
    console.log('Request URL: ' + req.url);
    return handler.handle(req);
  }
}
```
An interceptor defined like this, must be provided in a different way than before though.

Instead of providing it like this:
```
providers: [
  provideHttpClient(
    withInterceptors([loggingInterceptor]),
  )
],
```
You now must use withInterceptorsFromDi() and set up a custom provider, like this:
```
providers: [
  provideHttpClient(
    withInterceptorsFromDi()
  ),
  { provide: HTTP_INTERCEPTORS, useClass: LoggingInterceptor, multi: true }
]
```

# Управление формами
- template-driven (ngModule)
- reactive
```
@Component({
  selector: 'app-login',
  standalone: true,
  imports: [ReactiveFormsModule],
  templateUrl: '...',
  styleUrl: '...'
})
export class LoginComponent {
  form = new FormGroup({
    email: new FormControl(''),
    password: new FormControl(''),
    role: new FormControl<CustomType>(...)
  });

  onSubmit() {
    
  }
}

html:
...
<form [formGroup]="form">
  <input id="email" type="email" [formControl]="form.controls.email">
  либо
  <input id="email" type="email" formControlName="email">
</form>
```

Контроль массива:
```
source: new FormArray([
  new FormControl(false),
  new FormControl(false),
  new FormControl(false)
])

...
<fieldset formArrayName="source"> 
```

### Валидация
```
function mustContainQuestionMark(control: AbstractControl) {
  if (control.value.includes('?')) {
    return null;
  }

  return { doesNotContainQuestionMark: true };
}

function emailIsUnique(control: AbstractControl) {
  if (control.value !== 'test@example.com') {
    return of(null);
  }

  return of({ notUnique: true });
}

let initialEmailValue = '';
const savedForm = window.localStorage.getItem('saved-login-form');

if (savedForm) {
  const loadedForm = JSON.parse(savedForm);
  initialEmailValue = loadedForm.email;
}

@Component({
  selector: 'app-login',
  standalone: true,
  imports: [ReactiveFormsModule],
  templateUrl: './login.component.html',
  styleUrl: './login.component.css',
})
export class LoginComponent implements OnInit {
  private destroyRef = inject(DestroyRef);
  form = new FormGroup({
    email: new FormControl(initialEmailValue, {
      validators: [Validators.email, Validators.required],
      asyncValidators: [emailIsUnique],
    }),
    password: new FormControl('', {
      validators: [
        Validators.required,
        Validators.minLength(6),
        mustContainQuestionMark,
      ],
    }),
  });

  get emailIsInvalid() {
    return (
      this.form.controls.email.touched &&
      this.form.controls.email.dirty &&
      this.form.controls.email.invalid
    );
  }

  get passwordIsInvalid() {
    return (
      this.form.controls.password.touched &&
      this.form.controls.password.dirty &&
      this.form.controls.password.invalid
    );
  }

  const subscription = this.form.valueChanges
      .pipe(debounceTime(500))
      .subscribe({
        next: (value) => {
          window.localStorage.setItem(
            'saved-login-form',
            JSON.stringify({ email: value.email })
          );
        },
      });

    this.destroyRef.onDestroy(() => subscription.unsubscribe());
  }

  onSubmit() {
    console.log(this.form);
    const enteredEmail = this.form.value.email;
    const enteredPassword = this.form.value.password;
    console.log(enteredEmail, enteredPassword);
  }
}
```

# Перенаправление между страницами
```
<a href=""></a> - перезагружает страницу целиокм
<a routerLink=""></a> - меняет только router-outlet
```
### Взятие значения из ссылки
Если прописать
```
providers: [provideRouter, withComponentInputBinding()]
```
То можно брать значение из 'users/:userId' чезер Input() userId

Либо с помощью ActivatedRoute: this.activatedRoute.paramMap.subscribe((paramMap) => {
  const id = param.get('userId');
});
