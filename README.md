### Модификация работы с заявками 


#### 1. Обновление rules модели Request

`models/Request.php`:
```php
public function rules()
{
   return [
            [['address', 'phone', 'service_id', 'payment_id'], 'required'],
            [['service_id', 'payment_id', 'status_id'], 'integer'],
            [['custom_service'], 'string'],
            [['address'], 'string', 'max' => 255],
            ['phone', 'match', 'pattern' => '/^\+7\(\d{3}\)-\d{3}-\d{2}-\d{2}$/', 'message' => 'Телефон должен быть в формате +7(XXX)-XXX-XX-XX.'],
    ];
}
```

#### 2. Модификация страницы создания заявки

`views/request/_form.php`:
```php
 <?php $form = ActiveForm::begin(); ?>

    <?= $form->field($model, 'address')->textInput(['maxlength' => true])->label('Адрес') ?>

    <?= $form->field($model, 'phone')->textInput(['maxlength' => true])->label('Телефон') ?>

    <?= $form->field($model, 'service_id')->textInput()->dropDownList(
        ArrayHelper::map(Service::find()->all(), 'id', 'type'),
        ['prompt' => 'Выберите тип услуги']
    )->label('Услуга') ?>

    <?= $form->field($model, 'custom_service')->textarea(['rows' => 6])->label('Услуга') ?>

    <?= $form->field($model, 'payment_id')->textInput()->dropDownList(
        ArrayHelper::map(Payment::find()->all(), 'id', 'type'),
        ['prompt' => 'Выберите тип оплаты']
    )->label('Тип оплаты') ?>




    <div class="form-group">
        <?= Html::submitButton('Сохранить', ['class' => 'btn btn-success']) ?>
    </div>

    <?php ActiveForm::end(); ?>
```

![image](https://github.com/user-attachments/assets/d096aa84-3144-4405-abd5-4a33ade65bde)


---

#### 3. Модификация страницы просмотра заявок

`views/request/index.php`:
```php
$this->title = 'Заявки';
$this->params['breadcrumbs'][] = $this->title;
?>
<div class="request-index">

    <h1><?= Html::encode($this->title) ?></h1>

    <p>
        <?= Html::a('Создание заявки', ['create'], ['class' => 'btn btn-success']) ?>
    </p>

    <?php // echo $this->render('_search', ['model' => $searchModel]); ?>

    <?= GridView::widget([
        'dataProvider' => $dataProvider,
        'filterModel' => $searchModel,
        'columns' => [
            ['class' => 'yii\grid\SerialColumn'],

            [
                'attribute' => 'user_id',
                'value' => 'user.full_name',
                'label' => 'ФИО'
            ],
            [
                'attribute' => 'address',
                'label' => 'Адрес'
            ],
            [
                'attribute' => 'phone',
                'label' => 'Номер'
            ],
            [
                'attribute' => 'service_id',
                'value' => 'service.type',
                'label' => 'Услуга',
                'filter' => ArrayHelper::map(Service::find()->all(), 'id', 'type')
            ],
            [
                'attribute' => 'payment_id',
                'value' => 'payment.type',
                'label' => 'Оплата',
                'filter' => ArrayHelper::map(Payment::find()->all(), 'id', 'type')
            ],
            [
                'attribute' => 'status_id',
                'value' => 'status.type',
                'label' => 'Статус',
                'filter' => ArrayHelper::map(Status::find()->all(), 'id', 'type')
            ],
            
            [
            'attribute' => 'desired_date',
            'label' => 'Дата создания',
            'format' => ['datetime', 'php:d.m.Y']
            ],
            //'status_id',
            //'cancel_reason:ntext',
            [
                'class' => ActionColumn::className(),
                'urlCreator' => function ($action, Request $model, $key, $index, $column) {
                    return Url::toRoute([$action, 'id' => $model->id]);
                }
            ],
        ],
    ]); ?>


</div>

```

![image](https://github.com/user-attachments/assets/db8e4cae-2b41-45c8-8785-bbf0025f008d)


---

#### 4. Модификация контроллера

`controllers/RequestController.php`:
```php
public function actionCreate()
{
    $model = new Request();
    $model->user_id = Yii::$app->user->id; // Автоматически подставляем текущего пользователя

    if ($model->load(Yii::$app->request->post())) {
        if ($model->save()) {
            return $this->redirect(['view', 'id' => $model->id]);
        }
    }

    return $this->render('create', [
        'model' => $model,
    ]);
}
```

---

#### 5. Добавление динамического поведения (если выбрана "Иная услуга")

`views/request/_form.php` (после последней строчки):
```javascript
<?php
$js = <<<JS
$('#request-service_id').change(function() {
    if ($(this).val() == '3') { // ID для "Иная услуга"
        $('#request-custom_service').parent().show();
    } else {
        $('#request-custom_service').val('').parent().hide();
    }
}).trigger('change');
JS;
$this->registerJs($js);
?>
```

![image](https://github.com/user-attachments/assets/39aee01b-4dcb-4421-995b-353ca392000e)

![image](https://github.com/user-attachments/assets/1cd1bb07-618c-414d-88e4-f7fb0e6c3dc0)

---


#### 4. Модификация контроллера

Модифицируем внешний вид страницы просмотра заявки 
1. Откройте файл `views/requests/view.php`
2. Замените прямые поля на формат relationName.attributeName:

```php

<?php

use yii\helpers\Html;
use yii\widgets\DetailView;

/** @var yii\web\View $this */
/** @var app\models\Request $model */

$this->title = 'Заявка №' . $model->id;
$this->params['breadcrumbs'][] = ['label' => 'Заявки', 'url' => ['index']];
$this->params['breadcrumbs'][] = $this->title;
\yii\web\YiiAsset::register($this);
?>
<div class="request-view">

    <h1><?= Html::encode($this->title) ?></h1>

    <p>
        <?= Html::a('Update', ['update', 'id' => $model->id], ['class' => 'btn btn-primary']) ?>
        <?= Html::a('Delete', ['delete', 'id' => $model->id], [
            'class' => 'btn btn-danger',
            'data' => [
                'confirm' => 'Вы уверены что хотите удалить заявку?',
                'method' => 'post',
            ],
        ]) ?>
    </p>

    <?= DetailView::widget([
        'model' => $model,
        'attributes' => [
        
        [
            'attribute' => 'user_id',
            'value' => $model->user->full_name, 
            'label' => 'ФИО'
        ],
         [
            'attribute' => 'address',
            'label' => 'Адрес'
         ],
        [
            'attribute' => 'phone',
            'label' => 'Телефон'
         ],
        [
            'attribute' => 'service_id',
            'value' => $model->service->type, 
            'label' => 'Тип услуги',
            'visible' => empty($model->custom_service) // Показываем только если нет значения
        ],
        [
            'attribute' => 'custom_service',
            'label' => 'Тип услуги',
            'visible' => !empty($model->custom_service) // Показываем только если есть значение
        ],
        [
            'attribute' => 'desired_date',
            'label' => 'Дата создания',
            'format' => ['datetime', 'php:d.m.Y H:i'] // Красивое форматирование даты
        ],
        [
            'attribute' => 'payment_id',
            'value' => $model->payment->type, // предполагая relation getPaymentType()
            'label' => 'Способ оплаты'
        ],
        [
            'attribute' => 'status_id',
            'value' => $model->status->type,
            'label' => 'Статус'
        ],
        [
            'attribute' => 'cancel_reason',
            'visible' => $model->status_id == 3 // Показываем только для отмененных заявок
        ],
    ],
    ]) ?>

</div>

```

![image](https://github.com/user-attachments/assets/8c478c16-2408-4fa6-b89a-19930dbb3f5e)


--- 


### Итог

Мы:

1. Заменили текстовые поля на выпадающие списки
2. Настроили отображение связанных данных
3. Добавили динамическое поведение формы


### [Следующая лекция](https://github.com/petrocollege-web/9.-Admin-change-status)
### [Предыдущая лекция](https://github.com/petrocollege-web/7.-Menu-and-content)
