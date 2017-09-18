# Tản mạn
Gần đây dự án mình có xây dựng phần chat, sử dụng ActionCable của Rails. Trong quá trình tìm hiểu phần tài liệu, mình có đọc được 1 bài viết khá hay. Mặc dù đây không phải là bài viết về ứng dụng Chat nhưng ý tưởng và cách xây dựng khá thú vị, nên mình mạn phép dịch lại bài viết này để mọi người cùng tham khảo. 

Link bài viết: https://blog.codeship.com/realtime-with-react-and-rails/

# Realtime with React and Rails
Khi mình nghĩ về việc xây dựng 1 ứng dụng web sử dụng ActionCable(websocket) trong Rails app + React, đầu tiên mình nghĩ tới đó là xây dựng 1 ứng dụng Chat. Sau đó, mình lại nghĩ tới việc phát triển 1 realtime dashboard. Nhưng rồi mình nhận ra là mình không có dữ liệu khi người dùng chỉ có mỗi mình =)) Do đó, mình quyết định xây dựng một realtime map application để cho phép bạn có thể share vị trí hiện tại của mình tới bất kì ai mà bạn muốn.

Trong bài viết này, chúng ta sẽ sử dụng Rails, React (via react_on_rails gem), MobX và websockets (via ActionCable). The [demo app](https://tag-along-app.herokuapp.com/) có thể tìm thấy ở đây. Còn [đây](https://github.com/leighhalliday/tag-along) là full code repository.


## 1. Getting Started with a New Rails App
Chúng ta sẽ bắt đầu bằng việc generate 1 Rails app sử dụng command: `rails new tag_along --database=postgresql --webpack`. Sau đó, chúng ta add gem [react_on_rails](https://github.com/shakacode/react_on_rails) và chạy lệnh: `rails g react_on_rails:install`. Bạn chỉ cần nhớ là phải thêm tag: `<%= javascript_pack_tag 'webpack-bundle' %>` vào file application layout.

## 2. Creating Our Models
Trong application này, chúng ta sẽ làm việc với 2 model: `Trip`, có nhiệm vụ lưu trữ lại session tracking, còn `Checkin` model sẽ là lưu lại các vị trí và thời gian, được update liên tục. 

Trip model migration:
```Ruby
class CreateTrips < ActiveRecord::Migration[5.1]
  def change
    create_table :trips do |t|
      t.string :viewer_uuid, null: false
      t.string :owner_uuid, null: false
      t.string :name, null: false

      t.timestamps
    end

    add_index :trips, :viewer_uuid
    add_index :trips, :owner_uuid
  end
end
```

Trip model:

```
class Trip < ApplicationRecord
  has_many :checkins

  validates :viewer_uuid, :owner_uuid, :name, presence: true

  before_validation :set_uuids, on: :create

  def set_uuids
    self.viewer_uuid = SecureRandom.uuid
    self.owner_uuid = SecureRandom.uuid
  end
end
```

Checkins migration:
```
class CreateCheckins < ActiveRecord::Migration[5.1]
  def change
    create_table :checkins do |t|
      t.integer :trip_id, null: false
      t.decimal :lat, null: false, precision: 10, scale: 6
      t.decimal :lon, null: false, precision: 10, scale: 6
      t.datetime :captured_at, null: false

      t.timestamps
    end

    add_index :checkins, :trip_id
  end
end
```
Checkin model code:
```
class Checkin < ApplicationRecord
  belongs_to :trip
  validates :trip_id, :lat, :lon, :captured_at, presence: true
end
```

## 3. Organizing React
Bây giờ, chúng ta đã có model và các setup cơ bản, giờ là lúc chúng ta cần cấu trúc thư mục React. Chúng ta sẽ tạo 1 bundle mới, với tên `Trip` nằm bên trong thư mục `client`, với các subfolders:
* components: chưa code tương tác với MobX store
* containers: nơi để chứa các entry-points với MobX provider
* services: Chưa code tương tác với Rails API/sockets
* startup: đăng ký containers với ReactOnRails
* stores: MobX sẽ được đặt ở đây

Hãy nhớ phải update `client/ưebpack.config.js` file trong `entry` section để nó có thể nhận được bundle mới: `./app/bundles/Trip/startup/registration`

Chúng ta sẽ bắt đầu từ thư mục `startup`, nơi chỉ có 1 file duy nhất `registration.jsx`. Mục đích chính của file này đó là "đăng ký" bất cứ 1 component nào mà chúng ta muốn render với Rails view.

```
// client/app/bundles/Trip/startup/registration.jsx
import ReactOnRails from 'react-on-rails';

import NewTripContainer from '../containers/NewTripContainer';
import ViewTripContainer from '../containers/ViewTripContainer';

ReactOnRails.register({
  NewTripContainer,
  ViewTripContainer
});
```

## 4. Setting Up MobX
MobX là một tool quản lý trạng thái cho React. Đây là 1 lựa chọn thay thế cho Redux, cho phép bạn set up và sử dụng 1 cách rất đơn giản. Thêm vào đó, MobX có tính hướng đối tượng, khác hẳn so với tính functional thuần của Redux.

Chúng ta cần cài đặt `mobx` và `mobx-react` packages. Tiếp đó, chúng ta sẽ cần cài `babel-plugin-transform-decorators-legacy` và add plugin vào file `.babelrc`: `"plugins": ["transform-decorators-legacy"]`. Việc sử dụng decorators với MobX làm mọi thử trở nên rõ ràng và dễ dàng hơn rất nhiều.

Trong thư mục `stores`, chúng ta sẽ tạo `Tríptore`, nơi chúng ta sẽ quản lý trạng thái cho Trip và các Checkins của nó. Chúng ta cần làm rõ thuộc tính nào sẽ lưu trữ lại các trạng thái thay đổi bằng việc thêm chúng vào đầu của các class, sử dụng `@observable` decorator.

```
// client/app/bundles/Trip/stores/TripStore.js
import { observable, action } from 'mobx';
import TripApi from '../services/TripApi';

class TripStore {
  @observable trip = {};
  @observable checkins = [];

  constructor() {
    this.tripApi = new TripApi();
  }
}

const store = new TripStore();
export default store;
```

Có một chú ý quan trọng đó là store được export ở dòng cuối cùng của file: 1 biến instance của store. Chúng ta muốn 1 biến instance đơn của store sẽ được xử lý trong cả application.

Tiếp theo, chúng ta sẽ xử lý `NewTripContainer` component. Bằng việc đóng gói Prodiver xung quanh compoent, chúng ta có thể "inject" MobX store vào bất kì 1 phần tử con nào.

```
// client/app/bundles/Trip/containers/NewTripContainer.jsx
import React from 'react';
import { Provider } from 'mobx-react';
import TripStore from '../stores/TripStore';
import NewTrip from '../components/NewTrip';

export default (props, _railsContext) => {
  return (
    <Provider TripStore={TripStore}>
      <NewTrip {...props} />
    </Provider>
  );
};
```
Nếu chúng ta nhìn vào `NewTrip` component, không có nhiều thứ diễn ra ở đây. Chúng ta đơn giản include 2 phần thử con là có thể handle phần lớn công việc.

```
import React from 'react';
import TripForm from './TripForm';
import TripMap from './TripMap';

export default class NewTrip extends React.Component {
  render() {
    return (
      <div>
        <TripForm />
        <TripMap />
      </div>
    )
  }
}
```

Chúng ta sẽ render nó trong `views/trips/new` view với helper được cung cấp bởi `react_on_rails`. 

```
<%= react_component('NewTripContainer', props: {}, prerender: false) %>
```


## 5. Posting to Rails
Khi 1 page được load, chúng ta sẽ yêu cầu user nhập tên của họ, thứ chúng ta sẽ post vào Rails app. Trong Rails, chúng ta sẽ tạo mới 1 Trip vào database và trả về respond là đoạn JSON của model details. Phần này sẽ không sử dụng websockets. Nếu chúng ta bắt đầu từ cách nhìn của Rails, code sẽ có dạng: 

```
# app/controllers/trips_controller.rb
def create
  clean_old_trips
  trip = Trip.create!(trip_params)
  render json: trip.to_json
end
```

Nếu bạn đang phân vân `clean_old_trips` để làm gì, thì đó là 1 job để giữ database free trên Heroku luôn nhỏ :v . Tuy nhiên, ở đây chúng ta sẽ giả sử client gửi dữ liệu chuẩn xác và valid.

`TripForm` component sẽ được viết như ở dưới. Trong component đầu tiên, chúng ta sẽ "inject" TripStore vào trong. Điều đó có nghĩa chúng ta sẽ có 1 prop tên là `TripStore`, cho phép chúng ta gọi bất kì 1 action hoặc access vào bất kì thuộc tính observable nào mà chúng ta đã cài đặt ở trên. 

```
import React from 'react';
import { observer, inject } from 'mobx-react';

// Inject the TripStore into our component as a prop.
@inject('TripStore')
// Make our class "react" (re-render) to store changes.
@observer
export default class TripForm extends React.Component {
  // When user submits form, call the `createTrip` action, passing the name.
  handleSubmit = (e) => {
    e.preventDefault();
    const name = this.nameInput.value;
    this.props.TripStore.createTrip(name);
  }

  render() {
    const {TripStore} = this.props;

    // If we already have a trip in our store, display a link that can be
    // shared with anyone you want to share your realtime location with.
    if (TripStore.trip.name) {
      const trip_url = `${window.location.protocol}//${window.location.host}/trips/${TripStore.trip.viewer_uuid}`;

      return (
        <section className="trip-form-container">
          <p>
            Tracking <strong>{TripStore.trip.name}</strong>,
            share this link: <a href={trip_url}>{trip_url}</a>
          </p>
        </section>
      )
    }

    // Display the form allowing user to create a new Trip for themselves
    return (
      <section className="trip-form-container">
        
      </section>
    )
  }
}
```

Trong `TripStore`, chúng ta có thể thêm action được gọi ở trên, nó sẽ sử dụng API service chúng ta setup và subcribe realtime updates và gửi 1 realtime location tới server vào cùng thời điểm.

```
// client/app/bundles/Trip/stores/TripStore.js
@action createTrip = (name) => {
  this.tripApi.createTrip(name).
    then(trip => {
      // update our observable property, triggering re-render in component
      this.trip = trip;
      // subscribe to websocket channel for this specific "trip"
      this.tripApi.subscribeTrip(trip.viewer_uuid, checkin => {
        this.recordCheckin(checkin)
      });
      // send our location to server
      this.postCheckin();
    });
}
```

Trong API service, POST tới Rails server sẽ có dạng:

```
import ActionCable from 'actioncable';

export default class TripApi {
  constructor() {
    // for use later on in article when we talk about websockets
    this.cable = ActionCable.createConsumer('/cable');
    this.subscription = false;
  }

  createTrip = (name) => {
    return fetch('/trips', {
      method: 'post',
      headers: new Headers({
        'Content-Type': 'application/json'
      }),
      body: JSON.stringify({
        trip: {name}
      })
    }).
    then(response => response.json());
  }
}
```

## 6. ActionCable + Websockets
Khi làm việc với ActionCable trogn Rails, tôi phải add `redis` gem và update `config/cable.yml` để trỏ tới đúng Redis servers:
```
development:
  adapter: redis
  url: redis://localhost:6379/1
  channel_prefix: tag_along_development
production:
  adapter: redis
  url: <%= ENV['REDIS_URL'] %>
  channel_prefix: tag_along_production
```

Chúng ta sẽ tạo 1 file `trip_channel.rb` trong thư mục `app/channels`. Mỗi Channel trong ActionCable dùng để handle tập logic đơn, giống như Controller vậy.

Mình nhắc lại qua 1 vài định nghĩa:
* Channel: Giống như Controller trong ActionCable, nắm giữ các giao tiếp cho 1 use-case đơn.
* Room: Là nơi để bạn có thể gửi/ nhận các thông điệp realtime. Trong trường hợp của chúng ta, nó sẽ là 1 Trip cụ thể.
* Consumer: Đây là client. trong trường hợp này, browser sẽ connect tới server. Consumer có thể vừa nhận/ gửi thông tin qua kết nối websocket.
* Subcribe: Khi 1 Consumer kết nối tới server cho 1 Channel + Room cụ thể.
* Broadcast: Gửi thông tin tới tất cả subcribers của 1 Channel + Room cụ thể.

```
class TripChannel < ApplicationCable::Channel
  # called when user first subscribes
  # we can define where their information is "broadcast" from
  def subscribed
    stream_from "trip_#{params[:room]}"
  end

  # called when a Consumer sends information to the server
  def receive(data)
    # find trip using owner_uuid
    trip = Trip.find_by!(owner_uuid: data['owner_uuid'])

    # add additional checkin
    # not recording in demo to keep DB small on free Heroku
    # checkin = trip.checkins.create!({
    #   lat: data['lat'],
    #   lon: data['lon'],
    #   captured_at: Time.zone.at(data['captured_at'] / 1000)
    # })

    # broadcast checkin to subscribers
    ActionCable.server.broadcast("trip_#{params[:room]}", {
      lat: data['lat'],
      lon: data['lon'],
      captured_at: data['captured_at']
    })
  end
end
```

## 7. Receiving Realtime Data
Tiếp theo, để gửi dữ liệu qua websockets, chúng ta đầu tiên cần connect tới socket và subcribe vào 1 Channel + Room. Nếu bạn chú ý vào nơi `TripAPI` service trong JavaScript file, bạn sẽ thấy 1 dòng trong `constructor`: `this.cable = ActionCable.createConsumer('/cable');`. Đây là setup với 1 connection tới server như là 1 consumber của websocket này :D 

Sau đó, chúng ta sẽ xem hàm `subscribeTrip` trong TripApi, hàm mà được gọi bất cứ khi nào chúng ta muốn gửi/ nhận thông tin cho 1 Channel + Room. Chúng ta cung cấp 1 callback function, gọi mỗi lần server gửi dữ liệu realtime tới cho 1 Consumer.

```
// client/app/bundles/Trip/services/TripApi.js
subscribeTrip = (viewer_uuid, callback) => {
  this.subscription = this.cable.subscriptions.create({
    channel: "TripChannel",
    room: viewer_uuid
  }, {
    received: callback
  });
}
```

Hàm này được gọi bên trong của `createTrip` action function của `TripStore`:

```
// client/app/bundles/Trip/stores/TripStore.js
this.tripApi.subscribeTrip(trip.viewer_uuid, checkin => {
  this.recordCheckin(checkin)
});
```

Trong trường hợp này, thứ duy nhất mà server sẽ gửi cho chúng ta đó là "checkin" detail. 

```
// client/app/bundles/Trip/stores/TripStore.js
@action recordCheckin = (checkin) => {
  this.checkins.push({
    lat: parseFloat(checkin.lat),
    lon: parseFloat(checkin.lon),
    captured_at: parseInt(checkin.captured_at)
  });
  // Let's just keep last 25 checkins for performance
  this.checkins = this.checkins.slice(-25);
}
```

## 8. Sending Realtime Data
Chúng ta đã nói về làm thế nào để nhận realtime data, giờ chúng ta sẽ xem lại xem việc gửi realtime data sẽ làm như thế nào. Sau khi subcribe vào Channel + Room, chúng ta có 1 `subscription` object. Bây giờ, chúng ta sẽ hỏi user's location, cứ 2s 1 lần, gửi thông tin đó cho server sử dụng subscription.

```
// client/app/bundles/Trip/stores/TripStore.js
@action postCheckin = () => {
  // ask for location
  navigator.geolocation.getCurrentPosition(position => {
    // send location + owner_uuid (secret and only owner knows about it)
    this.tripApi.postCheckin(
      this.trip.owner_uuid,
      position.coords.latitude,
      position.coords.longitude,
      position.timestamp
    );

    // 2 seconds later do the whole thing again
    setTimeout(() => {
      this.postCheckin();
    }, 2000);
  });
}
```
Hàm `postChecking` có nhiệm vụ gửi thông tin qua websocket sử dụng subscription và hàm `send`: 
```
// client/app/bundles/Trip/services/TripApi.js
postCheckin = (owner_uuid, lat, lon, captured_at) => {
  this.subscription.send({
    owner_uuid,
    lat,
    lon,
    captured_at
  });
}
```

## 9. Showing Checkin Locations on the Map

Chúng ta đã xong phần gửi/ nhận dữ liệu từ server thông qua websocket và realtime update. Nhưng chúng ta sẽ làm gì với lượng thông tin về địa điểm này? Chúng ta sẽ show chung ra trên map! Để là điều này, chúng ta cần sử dụng `react-map-gl` package để làm việc với MapBox.

```
// client/app/bundles/Trip/components/TripMap.jsx
import React from 'react';
import { observer, inject } from 'mobx-react';
import MapGL, {Marker} from 'react-map-gl';
import moment from 'moment';
import MARKER_STYLE from '../markerStyle';

// You must sign up for a free access token
const token = process.env.MapboxAccessToken;

@inject('TripStore')
@observer
export default class TripStore extends React.Component {
  constructor() {
    super();

    this.state = {
      viewport: {
        latitude: 43.6532,
        longitude: -79.3832,
        // other viewport properties like width, height, zoom
      },
      settings: {
        // settings
      }
    };
  }

  // render a Marker for each checkin location
  renderMarker = (checkin) => {
    return (
      <Marker key={checkin.captured_at} longitude={checkin.lon} latitude={checkin.lat} >
        <div className="station">
          <span>{moment(checkin.captured_at).format('MMMM Do YYYY, h:mm:ss a')}</span>
        </div>
      </Marker>
    );
  }

  // Callback sent to the Map to handle dragging / panning / zooming of map
  onViewportChange = (viewport) => {
    this.setState({viewport});
  }

  // Helper function to set the viewport to the user's last checkin location
  viewport = () => {
    const {TripStore} = this.props;
    let latitude = 43.6532;
    let longitude = -79.3832;

    if (TripStore.checkins.length > 0) {
      const last = TripStore.checkins[TripStore.checkins.length - 1];
      latitude = last.lat;
      longitude = last.lon;
    }

    return {
      ...this.state.viewport,
      latitude,
      longitude
    };
  }

  render() {
    const {TripStore} = this.props;
    const viewport = this.viewport();

    // render actual map, mapping over each `checkins` in the TripStore
    return (
      <MapGL
        {...viewport}
        {...this.state.settings}
        mapStyle="mapbox://styles/mapbox/dark-v9"
        onViewportChange={this.onViewportChange}
        mapboxApiAccessToken={token} >
        <style>{MARKER_STYLE}</style>
        { TripStore.checkins.map(this.renderMarker) }
      </MapGL>
    );
  }
}
```

# Kết luận
Bài viết này bao gồm khá nhiều code, mặc dù vậy, chúng ta đã dựng lên được 1 app khá thú vị: 1 Realtime location-tracking map, cho phép chúng ta có thể share với bạn bè và gia định địa điểm mình đang ở. Chúng ta đã sử dụng khá nhiều công nghệ: React, MobX, ActionCable và MapBox.

Sử dụng ActionCable trong React không khác gì so với sử dụng nó với JS. Đêifu quan trọng nhất đó là chúng ta tạo được websocket connection để subcribe 1 Room + Channel. Từ đó, chúng ta có thể thoải mái gửi/ nhận thông tin. Việc xử lý thông tin thế nào đó là việc của bạn ;)

Qua bài viết này, hy vọng các bạn sẽ nắm được cách hoạt động, setup và sử dụng được ActionCable với React. 
Cảm ơn các bạn đã đọc bài.

Nguồn: https://blog.codeship.com/realtime-with-react-and-rails/












