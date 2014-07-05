# Başlangıç

> Role models are important.  <br/>
> -- Officer Alex J. Murphy / RoboCop

Bu rehberin amacı, Ruby on Rails 3 & 4 geliştirirken kullanabileceğiniz en iyi alışkanlıkları ve stil tavsiyelerini sunmaktır. Topluluğun katkısıyla hali hazırda oluşturulmuş olan [Ruby kodlama stili rehberi](https://github.com/bbatsov/ruby-style-guide) için tamamlayıcı bir rehberdir.

Tavsiyelerden bazıları sadece Rails 4.0+ için uygulanabilir.

[Transmuter](https://github.com/TechnoGate/transmuter) kullanarak bu rehberin PDF halini veya HTML kopyasını oluşturabilirsiniz.

Rehberin aşağıdaki dillere ait çevirileri bulunmaktadır:

* [Basit Çince](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhCN.md)
* [Geleneksel Çince](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhTW.md)
* [Rusça](https://github.com/arbox/rails-style-guide/blob/master/README-ruRU.md)

# Rails Stil Rehberi

Rails stil rehberi en iyi pratikleri önerdiğinden Rails programcıları tarafından yazılan kod diğer Rails programcıları tarafından devam ettirilebilir. Bir stil rehberi gerçek hayattaki alışıldık kullanımları gösterir ve insanlar tarafından reddedilen bir ideale bağlı kalır - Ne kadar iyi olduğuna bakılmaksızın alışılmadık risklerden kurtulmak için rehber gerekir.

Rehber, ilgili kuralların çeşitli bölümlerine ayrılmıştır. Kuralların arkasına gerekçelerini eklemeye çalıştım (Eğer eklenmemişse anlaşılır olduğunu varsaymışımdır).

Bu kuralları hiçbir yerden bulmadım - çoğu geniş yazılım mühendisliği kariyerime, Rails topluluk üyelerinden gelen geribildirim ve önerilere, ileri derece kabul edilen çeşitli Rails programlama kaynaklarına dayanmaktadır.

## İçindekiler

* [Yapılandırma (Configuration)](#yapılandırma-configuration)
* [Yönlendirme (Routing)](#yönlendirme-routing)
* [Denetleyiciler (Controllers)](#denetleyiciler-controllers)
* [Veri Modelleri (Models)](#veri-modelleri-models)
* [Göçler (Migrations)](#göçler-migrations)
* [Görünümler (Views)](#görünümler-views)
* [Uluslararasılaşma (Internationalization)](#uluslararasılaşma-internationalization)
* [Varlıklar (Assets)](#varlıklar-assets)
* [Mail Göndericileri (Mailers)](#mail-göndericileri-mailers)
* [Paket Yükleyicisi (Bundler)](#paket-yükleyicisi-bundler)
* [Hatalı Gemler (Flawed Gems)](#hatalı-gemler-flawed-gems)
* [İşlemleri Yönetme (Managing processes)](#işlemleri-yönetme-managing-processes)

## Yapılandırma (Configuration)

* Özel başlatma kodlarını `config/initializers` içine koyunuz. `initializers` içindeki kod uygulama başlatıldığında çalıştırılır.
* Her bir gem için başlatma kodlarını, gem ismiyle aynı olan farklı dosyalarda tutun, örneğin; `carrierwave.rb`,
  `active_admin.rb`, vb.
* Geliştirme (development), test ve üretim (production) ortamları için uygun ayarları düzenleyin (`config/environments/` dizini altındaki ilişkili dosyalarda)
  * Ön derleme için ek varlıkları (assets) belirtin (eğer varsa):

        ```Ruby
        # config/environments/production.rb
        # Ek varlıkları (assets) ön derle (application.js, application.css, ve JS/CSS olmayan bütün dosyalar zaten eklenmiştir)
        config.assets.precompile += %w( rails_admin/rails_admin.css rails_admin/rails_admin.js )
        ```

* `config/application.rb` dosyası içerisindeki tüm ortamlara uygulanabilir yapılandırma tutun.
* `production` ortamına hemen hemen benzeyen ek `staging` ortamı oluşturun.

## Yönlendirme (Routing)

* [RESTful](https://www.google.com.tr/#q=restful+nedir) kaynağına daha fazla eylemler(actions) eklemeye ihtiyacınız varsa (Gerçekten onlara ihtiyacınız var mı?) `member` ve `collection` yollarını kullanın.

    ```Ruby
    # kötü
    get 'subscriptions/:id/unsubscribe'
    resources :subscriptions

    # iyi
    resources :subscriptions do
      get 'unsubscribe', on: :member
    end

    # kötü
    get 'photos/search'
    resources :photos

    # iyi
    resources :photos do
      get 'search', on: :collection
    end
    ```

* Eğer çoklu `member/collection` yolları tanımlamanız gerekiyorsa başka blok sözdizimi kullanın.

    ```Ruby
    resources :subscriptions do
      member do
        get 'unsubscribe'
        # daha fazla yol
      end
    end

    resources :photos do
      collection do
        get 'search'
        # daha fazla yol
      end
    end
    ```

* ActiveRecord veri modelleri arasındaki ilişkiyi daha iyi göstermek için iç içe yolları kullanın.

    ```Ruby
    class Post < ActiveRecord::Base
      has_many :comments
    end

    class Comments < ActiveRecord::Base
      belongs_to :post
    end

    # routes.rb
    resources :posts do
      resources :comments
    end
    ```

* İlgili eylemleri gruplamak için isim alanlı (namespaced) yolları kullanın.

    ```Ruby
    namespace :admin do
      # Directs /admin/products/* to Admin::ProductsController
      # (app/controllers/admin/products_controller.rb)
      resources :products
    end
    ```

* Eski dağınık denetleyici yollarını asla kullanmayın. Bu yol her bir denetleyicideki tüm eylemleri GET isteği ile ulaşılabilir yapacaktır. 

    ```Ruby
    # çok kötü
    match ':controller(/:action(/:id(.:format)))'
    ```

* Yolları tanımlamak için `match` kullanmayın. Rails 4'te bu kaldırılmıştır.

## Denetleyiciler (Controllers)

* Denetleyicileri zayıf tutunuz - denetleyiciler sadece görünüm katmanı için verileri almalıdır ve hiç bir iş mantığı (business logic) içermemelidir (bütün iş mantıkları doğal olarak model içerisinde yer almalıdır).
* İdeal olarak her bir denetleyici eylemi find veya new methodlarından başka sadece bir methodu çağırmalıdır.
* Denetleyici (controller) ve görünüm (view) arasında iki değişkenden fazla değişken paylaşmayınız.

## Veri Modelleri (Models)

* ActiveRecord olmayan model sınıflarını serbestçe ekleyebilirsiniz.
* Veri modellerine kısaltma kullanmadan anlamlı (fakat kısa) isimler veriniz.
* Eğer ActiveRecord davranışını destekleyen veri modeli nesnelerine ihtiyacınız varsa (doğrulama gibi) 
  [ActiveAttr](https://github.com/cgriego/active_attr) gemini kullanınız.

    ```Ruby
    class Message
      include ActiveAttr::Model

      attribute :name
      attribute :email
      attribute :content
      attribute :priority

      attr_accessible :name, :email, :content

      validates :name, presence: true
      validates :email, format: { with: /\A[-a-z0-9_+\.]+\@([-a-z0-9]+\.)+[a-z0-9]{2,4}\z/i }
      validates :content, length: { maximum: 500 }
    end
    ```

    Daha kapsamlı örnekler için
    [RailsCast'teki ilgili konuya](http://railscasts.com/episodes/326-activeattr) bakınız.

### Etkin Kayıt (ActiveRecord)

* İyi bir nedeniniz olmadığı sürece (veri tabanı kontrolünüzde değilse) ActiveRecord varsayılanlarını değiştirmekten kaçınınız (tablo isimleri,birincil anahtar vb.).

    ```Ruby
    # kötü - şemayı değiştirebiliyorsanız bunu yapmayınız 
    class Transaction < ActiveRecord::Base
      self.table_name = 'order'
      ...
    end
    ```

* Makro tarzı methodları (`has_many`, `validates` vb.) sınıf tanımının başlangıcında gruplayınız.

    ```Ruby
    class User < ActiveRecord::Base
      # ilk default scope kullanın  (varsa)
      default_scope { where(active: true) }

      # daha sonra sabitleri ekleyin
      GENDERS = %w(male female)

      # sonrasında attr ile ilişkili makroları koyun
      attr_accessor :formatted_date_of_birth

      attr_accessible :login, :first_name, :last_name, :email, :password

      # daha sonra ilişki makroları
      belongs_to :country

      has_many :authentications, dependent: :destroy

      # ve doğrulama makrolarını ekleyin
      validates :email, presence: true
      validates :username, presence: true
      validates :username, uniqueness: { case_sensitive: false }
      validates :username, format: { with: /\A[A-Za-z][A-Za-z0-9._-]{2,19}\z/ }
      validates :password, format: { with: /\A\S{8,128}\z/, allow_nil: true}

      # sırada geri çağrımlar var
      before_save :cook
      before_save :update_username_lower

      # diğer makrolar (devise'ın vb.) geri çağrımlardan sonra eklenmelidir

      ...
    end
    ```

* `has_and_belongs_to_many` yerine `has_many :through` tercih edin. `has_many:through` kullanmak birleştirme modellerinde ek özelliklere ve doğrulamalara izin verir.

    ```Ruby
    # has_and_belongs_to_many kullanmak
    class User < ActiveRecord::Base
      has_and_belongs_to_many :groups
    end

    class Group < ActiveRecord::Base
      has_and_belongs_to_many :users
    end

    # tercih edilen yol - has_many :through kullanmak
    class User < ActiveRecord::Base
      has_many :memberships
      has_many :groups, through: :memberships
    end

    class Membership < ActiveRecord::Base
      belongs_to :user
      belongs_to :group
    end

    class Group < ActiveRecord::Base
      has_many :memberships
      has_many :users, through: :memberships
    end
    ```

* `read_attribute(:attribute)` yerine `self[:attribute]` kullanın .

    ```Ruby
    # kötü
    def amount
      read_attribute(:amount) * 100
    end

    # iyi
    def amount
      self[:amount] * 100
    end
    ```

* Her zaman yeni
  ["çekici" doğrulamaları](http://thelucid.com/2010/01/08/sexy-validation-in-edge-rails-rails-3/) kullanın.

    ```Ruby
    # kötü
    validates_presence_of :email

    # iyi
    validates :email, presence: true
    ```

* Özel doğrulamalar birden fazla kullanıldığında veya doğrulama, bazı düzenli ifadeleri (regular expression) eşleştirmekse, özel bir doğrulayıcı dosya oluşturun.

    ```Ruby
    # kötü
    class Person
      validates :email, format: { with: /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i }
    end

    # iyi
    class EmailValidator < ActiveModel::EachValidator
      def validate_each(record, attribute, value)
        record.errors[attribute] << (options[:message] || 'is not a valid email') unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
      end
    end

    class Person
      validates :email, email: true
    end
    ```

* Özel doğrulayıcıları `app/validators` altında tutun.
* Eğer ilgili birçok uygulamayı sürdürüyorsanız veya doğrulayıcılarınız yeteri kadar kapsamlıysa, özel doğrulayıcılarızı paylaşılan bir geme çıkarmayı düşünün.
* İsimli alanları (named scopes) serbestçe kullanın.

    ```Ruby
    class User < ActiveRecord::Base
      scope :active, -> { where(active: true) }
      scope :inactive, -> { where(active: false) }

      scope :with_orders, -> { joins(:orders).select('distinct(users.id)') }
    end
    ```

* Eğer isimli alanların (named scopes) ihtiyaç duyulduğunda yüklenmesini istiyorsanız (lazy initialization) `lambdalar` ile kullanın  (Rails 3 için bir tavsiyedir, fakat Rails 4 için zorunludur).

    ```Ruby
    # kötü
    class User < ActiveRecord::Base
      scope :active, where(active: true)
      scope :inactive, where(active: false)

      scope :with_orders, joins(:orders).select('distinct(users.id)')
    end

    # iyi
    class User < ActiveRecord::Base
      scope :active, -> { where(active: true) }
      scope :inactive, -> { where(active: false) }

      scope :with_orders, -> { joins(:orders).select('distinct(users.id)') }
    end
    ```

* İsimli alanlar lambda ile tanımlandığında ve parametreler çok karışık olmaya başladığında, isimli alanla aynı amacı gerçekleştiren ve geriye `ActiveRecord::Relation` nesnesi döndüren bir sınıf methodu tercih edilir. Bu şekilde daha basit alanlar bile oluşturabilirsiniz.

    ```Ruby
    class User < ActiveRecord::Base
      def self.with_orders
        joins(:orders).select('distinct(users.id)')
      end
    end
    ```

* [`update_attribute`](http://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-update_attribute) methodunun kullanımından sakının. Veri modelleri doğrulamalarını çalıştırmaz (`update_attributes` olmadan) ve veri modelinin durumunu bozabilir.
* Kullanıcı dostu URL kullanın. URL'lerde veri modelinin `id`'si yerine tanımlayıcı özelliğini gösterin.
Bunu yapmak için birden fazla yol var:
  * Veri modelinin `to_param` methodunu ezerek. Bu method nesnelere URL oluşturmak için kullanılır.
  Varsayılan yürütmelerde kaydın `id`'si String olarak döndürülür. Bu daha okunabilir özellik içermek amacıyla ezilebilir.

        ```Ruby
        class Person
          def to_param
            "#{id} #{name}".parameterize
          end
        end
        ```

    Bunu kullanıcı dostu bir değere dönüştürmek için, `parameterize` methodu çağrılmalıdır. ActiveRecord `find` methodu ile nesnenin bulunabilmesi için başlangıçta nesnenin `id`'si olmalıdır.

  * `friendly_id` gemini kullanın. Veri modelinin `id`si yerine daha okunabilir URL'ler üretmenize yardımcı olur.

        ```Ruby
        class Person
          extend FriendlyId
          friendly_id :name, use: :slugged
        end
        ```

  `friendly_id` geminin kullanımıyla ilgili daha fazla bilgi almak içim [gem dökümantasyonuna](https://github.com/norman/friendly_id) bakınız.

* ActiveRecord nesnelerinin yığınları üzerinde öteleme yapmak için `find_each` methodunu kullanın. Bu durumda, yığın işleme methodları yığındaki kayıtlarla çalışmanıza izin verir, böylece hafıza tüketimi önemli ölçüde azalır.Veritabanından kayıtların yığını üzerinde döngü yapmak bütün nesneleri bir kerede klonlamayı deneyeceğinden verimsizdir (`all` methodu kullanmak gibi). 

    ```Ruby
    # kötü
    Person.all.each do |person|
      person.do_awesome_stuff
    end

    Person.where("age > 21").each do |person|
      person.party_all_night!
    end

    # iyi
    Person.all.find_each do |person|
      person.do_awesome_stuff
    end

    Person.where("age > 21").find_each do |person|
      person.party_all_night!
    end
    ```

## Göçler (Migrations)

* `schema.rb` (veya `structure.sql`) dosyasını versiyon kontrolü altında tutun.
* Boş bir veritabanını başlangıç durumuna getirmek için `rake db:migrate` yerine `rake db:schema:load` kullanın.
* Veritabanı test şemasını güncellemek için `rake db:test:prepare` kullanın.
* Varsayılan değerleri uygulama katmanı yerine göçlerin (migrations) içerisinde yürütün.

    ```Ruby
    # kötü - uygulama varsayılan değeri yürüttü
    def amount
      self[:amount] or 0
    end
    ```

    Bazı Rails geliştiricileri tarafından sadece Rails içerisinde tablo varsayılanlarının uygulanması önerilmektedir, fakat birçok uygulama hatasına karşı verileriniz savunmasız kalacağından bu aşırı kırılgan bir yaklaşımdır. Basit olmayan uygulamaların çoğu veritabanını başka uygulamalarla paylaşmaktadır, bu nedenle veri bütünlüğünü etkilemek imkansızdır.

* Yabancı anahtar kısıtlamalarını (foreign-key constraints) kullanın. ActiveRecord'un bunları desteklememesine karşın
[schema_plus](https://github.com/lomba/schema_plus) ve [foreigner](https://github.com/matthuhiggins/foreigner) gibi 3.parti gemler bulunmaktadır.

* Yapıcı göçleri yazıyorken (tablo ve sütun eklerken), göç yapmak için Rails 3.1deki yeni yolu kullanın  - `up` ve `down` methodları yerine `change` methodunu kullanın.


    ```Ruby
    # eski yol
    class AddNameToPeople < ActiveRecord::Migration
      def up
        add_column :people, :name, :string
      end

      def down
        remove_column :people, :name
      end
    end

    # tercih edilen yeni yol
    class AddNameToPeople < ActiveRecord::Migration
      def change
        add_column :people, :name, :string
      end
    end
    ```

* Veri modeli sınıflarını göçlerde kullanmayın. Veri modeli sınıfları sıkça değişime uğrar ve gelecek göçlerdeki bazı noktalarda veri modelindeki bu değişiklikler nedeniyle çalışma durabilir.

## Görünümler (Views)

* Bir görünümden model katmanını direk olrak çağırmayınız.
* Görünümlerde karışık biçimleme yapmayınız, biçimlemeyi görünüm yardımcısındaki (view helper) bir methoda veya veri modeline aktarınız.
* Kod tekrarlarını kısmi şablon (partial templates) ve düzenler (layouts) kullanarak azaltınız.

## Uluslararasılaşma (Internationalization)

* Stringleri veya diğer yerel ayarları görünüm, veri modeli ve denetleyiciler içinde kullanmayınız. Bu metinler `config/locales` dizinindeki yerel dosyalara (locale files) taşınmalıdır.
* ActiveRecord veri modelindeki etiketlerin tercüme edilmesine gerek duyulduğunda, `activerecord` alanını kullanın:

    ```
    en:
      activerecord:
        models:
          user: Member
        attributes:
          user:
            name: "Full name"
    ```

    Sonrasında `User.model_name.human` "Member" döndürecek ve `User.human_attribute_name("name")` "Full name" döndürecektir. Özelliklerin bu tercümeleri görünüm içerisinde etiket olarak kullanılacaktır.

* ActiveRecord özelliklerinin çevirilerinden görünüm içerisinde kullanılan metinleri ayırın. Veri modelleri için yerel dosyaları `models` klasörüne ve görünüm içerisinde kullanılan metinleri `views` klasörüne yerleştirin.
  * Ek dizinlerle birlikte yerel dosyaların düzeni tamamlandığında, bu dizinler dosyanın yüklenebilmesi için `application.rb` içerisinde belirtilmelidir.

        ```Ruby
        # config/application.rb
        config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}')]
        ```

* Tarih ve para formatları gibi ortak kullanılan yerelleştirme ayarlarını temel `locales` dizini altındaki dosyalarda tutunuz.
* I18n methodlarının kısa kalıplarını kullanın:  `I18n.translate` yerine `I18n.t`
ve `I18n.localize` yerine `I18n.l` .
* Görünümlerdeki metinler için tembel arama (lazy search) kullanın. Aşağıdaki yapımız olduğunu varsayalım:

    ```
    en:
      users:
        show:
          title: "User details page"
    ```

    `users.show.title` değeri için 
    `app/views/users/show.html.haml` şablonunda böyle arama yapılabilir:

    ```Ruby
    = t '.title'
    ```

* `:scope` ayarını belirtmek yerine denetleyicilerde ve veri modellerinde nokta ile ayrılmış anahtarlar kullanın. Nokta ile ayrılmış çağrılar, okumak ve hiyerarşiyi izlemek için daha kolay bir yapıdadır.

    ```Ruby
    # bu çağrıyı kullanın
    I18n.t 'activerecord.errors.messages.record_invalid'

    # bunun yerine
    I18n.t :record_invalid, :scope => [:activerecord, :errors, :messages]
    ```

* Rails i18n ile ilgili daha fazla detaylı bilgi [Rails
Rehberinde]
(http://guides.rubyonrails.org/i18n.html) bulunabilir

## Varlıklar (Assets)

Uygulamanızdaki düzeni kontrol etmek için [assets pipeline](http://guides.rubyonrails.org/asset_pipeline.html) kullanın.

* `app/assets` klasörünü özel stil sayfaları (stylesheets), javascript dosyaları ve resimler için kullanınız.
* Uygulama alanına koymaya elverişli olmayacak kendi kütüphanelerinizi `lib/assets` klasörüne atınız.
*  [jQuery](http://jquery.com/) veya [bootstrap](http://twitter.github.com/bootstrap/) gibi 3.parti kodlar `vendor/assets` içerisine yerleştirilmelidir.
* Eğer mümkünse, varlıkların (assets) (e.g. [jquery-rails](https://github.com/rails/jquery-rails), [jquery-ui-rails](https://github.com/joliss/jquery-ui-rails), [bootstrap-sass](https://github.com/thomas-mcdonald/bootstrap-sass), [zurb-foundation](https://github.com/zurb/foundation) gibi)  gem hallerini kullanınız.

## Mail Göndericileri (Mailers)

* Mail göndericilerine `SomethingMailer` adını verin. Mailer son eki olmadan neyin mail göndericisi olduğu ve hangi görünümlerin bu mail göndericisiyle ilgili olduğu tam belli olmaz.
* Hem HTML hem de yalın metin görünüm şablonları (plain-text view templates) kullanın.
* Geliştirme ortamınızda mail gönderirken oluşan hataları etkinleştirin. Hatalar varsayılan olarak devre dışıdır.

    ```Ruby
    # config/environments/development.rb

    config.action_mailer.raise_delivery_errors = true
    ```

* Geliştirme ortamında [Mailcatcher](https://github.com/sj26/mailcatcher) gibi yerel bir SMTP sunucusu kullanın.

    ```Ruby
    # config/environments/development.rb

    config.action_mailer.smtp_settings = {
      address: 'localhost',
      port: 1025,
      # daha fazla ayar
    }
    ```

* Ana makine adları için varsayılan ayarları sağlayın.

    ```Ruby
    # config/environments/development.rb
    config.action_mailer.default_url_options = { host: "#{local_ip}:3000" }


    # config/environments/production.rb
    config.action_mailer.default_url_options = { host: 'your_site.com' }

    # mail göndericisi sınıfınızda
    default_url_options[:host] = 'your_site.com'
    ```

* Eğer email içerisinde sitenize yönlendiren bir bağlantı kullanmak istiyorsanız , `_path` methodu yerine her zaman `_url` methodlarını kullanın. `_url` methodları site adını içerirken  `_path` methodları içermez.

    ```Ruby
    # yanlış
    Bu yolla ilgili her zaman daha fazla bilgi bulabilirsiniz
    = link_to 'here', url_for(course_path(@course))

    # doğru
    Bu yolla ilgili her zaman daha fazla bilgi bulabilirsiniz
    = link_to 'here', url_for(course_url(@course))
    ```

* Mailin geldiği ve gittiği adresleri düzgünce ayarlayın. Aşağıdaki düzeni kullanın:

    ```Ruby
    # mail göndericisi sınıfı içerisinde
    default from: 'Your Name <info@your_site.com>'
    ```

* Test ortamlarınız için e-mail gönderim methodlarının `test` olarak ayarlandığına emin olun:

    ```Ruby
    # config/environments/test.rb

    config.action_mailer.delivery_method = :test
    ```

* Geliştirme ve üretim ortamları için gönderim methodu `smtp` olmalıdır:

    ```Ruby
    # config/environments/development.rb, config/environments/production.rb

    config.action_mailer.delivery_method = :smtp
    ```

* Html e-mailler gönderirken bütün stiller satır içinde olmalıdır (Her ne kadar kod tekrarının önüne geçmeyi ve sürdürülebilirliğini daha da zor bir hale getirse de), bazı mail istemcilerinde harici stiller problem yaratabilmektedir . Stilleri değiştiren ve onları ilgili html etiketlerine koyan iki tane benzer gem vardır:
  [premailer-rails](https://github.com/fphilipe/premailer-rails) ve
  [roadie](https://github.com/Mange/roadie).

* Sayfa isteği üretilirken mail göndermekten kaçınılmalıdır. Bu, sayfalar yüklenirken gecikmeye ve çoklu mail gönderiliyorsa isteklerin zaman aşımına uğramasına neden olur. Bunun üstesinden gelmek için emailler [sidekiq](https://github.com/mperham/sidekiq) gemi yardımıyla arka planda gönderilebilir.

## Paket Yükleyicisi (Bundler)

* Sadece geliştirme veya sadece test için kullanılan gemleri Gemfile içerisinde uygun gruplara koyun.
* Projenizde sadece kurulu gemleri kullanın. Eğer az bilinen bazı gemleri kullanmayı düşünüyorsanız, onların kaynak kodlarını önce dikkatlice incelemelisiniz.
* İşletim sistemlerine özgü gemler farklı işletim sistemleri kullanan çoklu geliştiricilerin olduğu projeler için `Gemfile.lock` dosyasını  sürekli değiştirecektir.
Gemfile içerisinde tüm OS X'e özgü gemleri `darwin` grubuna ve tüm Linux'a özgü gemleri linux grubuna ekleyin:

    ```Ruby
    # Gemfile
    group :darwin do
      gem 'rb-fsevent'
      gem 'growl'
    end

    group :linux do
      gem 'rb-inotify'
    end
    ```

    Doğru ortamlarda uygun gemleri çağırmak için, `config/application.rb` dosyasına aşağıdaki kodu ekleyin:

    ```Ruby
    platform = RUBY_PLATFORM.match(/(linux|darwin)/)[0].to_sym
    Bundler.require(platform)
    ```

* Versiyon kontrolünden `Gemfile.lock` dosyasını kaldırmayın. Bu rastgele üretilen bir dosya değildir, tüm takım arkadaşlarınızın `bundle install` yaptığında aynı gem versiyonlarını aldığına emin olur.

## Hatalı Gemler (Flawed Gems)

Bu problemli olan veya yerini başka gemler alan gemlerin bir listesidir. Projelerinizde bunları kullanmaktan kaçınmalısınız.

* [rmagick](http://rmagick.rubyforge.org/) - bu gem hafıza tüketimiyle kötü üne sahiptir. Yerine
[minimagick](https://github.com/probablycorey/mini_magick) kullanın.
* [autotest](http://www.zenspider.com/ZSS/Products/ZenTest/) - testleri otomatik çalıştırmak için eski çözümdür. [guard](https://github.com/guard/guard) ve [watchr](https://github.com/mynyml/watchr) gemlerinden bayağı aşağıdadır.
* [rcov](https://github.com/relevance/rcov) - kod test aracıdır, Ruby 1.9 ile uyumlu değildir. Yerine
  [SimpleCov](https://github.com/colszowka/simplecov) kullanın.
* [therubyracer](https://github.com/cowboyd/therubyracer) - Bu gemin üretimde kullanılması hafızanın geniş bir kısmını kullandığından son derece bezdiricidir. Bunun yerine `node.js` kullanmanızı öneriyorum.

Bu liste hala geliştirilmektedir. Eğer başka popüler ve hatalı gemler biliyorsanız, lütfen bizi bilgilendirin.

## İşlemleri Yönetme (Managing processes)

* Eğer projeleriniz çeşitli harici işlemlere bağlıysa onları yönetmek için 
  [foreman](https://github.com/ddollar/foreman) gemini kullanın.

# İlave Kaynaklar

Rails stili üzerine birçok güzel kaynak bulunmaktadır, eğer zamanınız varsa bakmanızı öneriyoruz:

* [The Rails 4 Way](http://www.amazon.com/The-Rails-Addison-Wesley-Professional-Ruby/dp/0321944275)
* [Ruby on Rails Guides](http://guides.rubyonrails.org/)
* [The RSpec Book](http://pragprog.com/book/achbd/the-rspec-book)
* [The Cucumber Book](http://pragprog.com/book/hwcuc/the-cucumber-book)
* [Everyday Rails Testing with RSpec](https://leanpub.com/everydayrailsrspec)

# Katkıda Bulunmak

Bu rehberde yazılan hiçbir şey değiştirilemez değildir.Benim arzum Rails kodlama stili ile ilgili herkesin birlikte çalışmasıdır. Bu sayede, tüm Ruby topluluğu için oldukça yararlı bir kaynak yaratabiliriz.

Yaptığınız düzeltmelerle ilgili rahatlıkla [pull request](https://help.github.com/articles/using-pull-requests) gönderebilirsiniz veya konu açabilirsiniz. Şimdiden yardımlarınız için teşekkürler! 

Ayrıca [gittip](https://www.gittip.com/bbatsov)'e yapacağınız finansal katkılarla projeyi (ve RuboCop) destekleyebilirsiniz .

[![Gittip ile katkıda bulunmak](https://rawgithub.com/twolfson/gittip-badge/0.2.0/dist/gittip.png)](https://www.gittip.com/bbatsov)

## Nasıl katkıda bulunurum?

Çok kolay, sadece [destek kurallarını](https://github.com/bbatsov/rails-style-guide/blob/master/CONTRIBUTING.md) takip ediniz.

# Lisans

![Creative Commons Lisansı](http://i.creativecommons.org/l/by/3.0/88x31.png)
Bu çalışma [Creative Commons Attribution 3.0 Unported Lisansı](http://creativecommons.org/licenses/by/3.0/deed.en_US) altında lisanslıdır.

# Duymayan Kalmasın

Topluluğun katkısıyla oluşturulan bu rehberin olduğundan habersiz olunması kullanımın az olmasına sebep olur. Rehberle ilgili tweet atın, arkadaşlarınızla paylaşın. Her yorum, öneri veya düşünce bu rehberi biraz daha iyi hale getirecektir. Ve bizler mümkün olabilecek en iyi rehberi istiyoruz, değil mi?

Teşekkürler,<br/>
[Bozhidar](https://twitter.com/bbatsov)
