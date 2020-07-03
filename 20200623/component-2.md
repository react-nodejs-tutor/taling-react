지금까지 배운 내용들로 새로운 프로젝트를 진행해볼게요.

```js
// skeleton code를 git clone해주세요

git clone https://github.com/react-nodejs-tutor/football-api-prac-demo.git

// package.json에 있는 라이브러리들을 설치해주세요

yarn add

yarn start
```

우선 이번 프로젝트를 진행하기 위해서는 apifootball 사이트에 가입하고 apiKey를 발급받으셔야 합니다.

```js
https://apifootball.com에 들어가서 register하고 apiKey 발급받기
```

clone한 폴더 구조를 한번 살펴보세요.

이전에 진행한 프로젝트에 비해서 많이 복잡해보이죠?

하지만 그렇지 않습니다. 파일이 보이지 않게 폴더를 접고 폴더명만 살펴보세요.

관련있는 파일들끼리 폴더로 구조화해놓았기 때문에 이렇게 하면 눈에 쉽게 들어오실 것입니다.

이번 프로젝트 구조는 다음과 같습니다.

![](/assets/app.png)

우선 Calendar와 League는 신경쓰지말고 유저가 url에 처음 접속했을때 영국 프리미어리그 2020년 1월 1일 ~ 2020년 2월 1일 경기목록 및 스코어를 보여주는 작업을 해볼까요?

api요청할때는 발급받은 개인 API KEY를 넣어주세요.

```js
// MatchList.js

import React, { Component } from 'react';
import axios from 'axios';

class MatchList extends Component {
    state = {
        data: null
    };

    getData = async () => {
        try {
            const response = await axios.get(
                'https://apiv2.apifootball.com/?action=get_events&from=2020-01-01&to=2020-02-01&league_id=148&APIkey=<본인API_KEY>'
            );

            console.log(response.data);
            this.setState({
                data: response.data
            });
        } catch (e) {
            console.error(e);
        }
    };

    componentDidMount() {
        this.getData();
    }

    render() {
        return <div></div>;
    }
}

export default MatchList;
```

여기까지 하고 console을 한번 보세요.

cdm에서 데이터를 잘가져오고 있나요?

그러면 값을 확인하고 어떤값을 key로 써야할지도 살펴보세요!

이번엔 불러온 데이터들을 보여줍시다.

```js
// MatchList.js

import React, { Component } from 'react';
import axios from 'axios';

class MatchList extends Component {
    state = {
        data: null
    };

    getData = async () => {
        try {
            const response = await axios.get(
                'https://apiv2.apifootball.com/?action=get_events&from=2020-01-01&to=2020-02-01&league_id=148&APIkey=<본인API_KEY>'
            );

            console.log(response.data);
            this.setState({
                data: response.data
            });
        } catch (e) {
            console.error(e);
        }
    };

    componentDidMount() {
        this.getData();
    }

    render() {
        const { data } = this.state;
        return <div>{data && data.map(d => <Match key={d.match_id} data={d} />)}</div>;
    }
}

export default MatchList;
```

인터넷이 느린 사용자는 처음에 들어왔을때 수 ms동안 아무내용도 표시되지 않기때문에 페이지를 이탈할 가능성이 있습니다.

따라서 오류가 난 것이 아니라고 말해주기 위해 '데이터를 불러오는 중이다'라는 메세지를 띄워줍시다.

```js
// MatchList.js

import React, { Component } from 'react';
import axios from 'axios';

class MatchList extends Component {
    state = {
        loading: false
        data: null
    };

    getData = async () => {
        try {
            this.setState({
                loading: true
            })

            const response = await axios.get(
                'https://apiv2.apifootball.com/?action=get_events&from=2020-01-01&to=2020-02-01&league_id=148&APIkey=<본인API_KEY>'
            );

            console.log(response.data);
            this.setState({
                data: response.data
            });
        } catch (e) {
            console.error(e);
        }

        this.setState({
            loading: false
        })
    };

    componentDidMount() {
        this.getData();
    }

    render() {
        const { loading, data } = this.state;
        return (
            <div>
                {loading && <h3 style={{ textAlign: 'center' }}>데이터를 불러오는중입니다...</h3>}
                {!loading && data && data.map(d => <Match key={d.match_id} data={d} />)}
            </div>
        );
    }
}

export default MatchList;
```

이제 유저가 날짜를 선택하면 해당 날짜 범위에 있는 경기 리스트 및 스코어를 불러오게 하겠습니다.

제가 작성한 스켈레톤 코드에서는 이미 calendar컴포넌트가 자체적으로 state를 관리하고 있습니다.

그렇다면 날짜가 바뀌면 바뀐대로 새로운 주소가 api call을 날려줘야겠죠?

따라서 component\(1\)에서 했던 것처럼 최상단 app에서 해당 날짜를 별도의 state로 받아서 다시 MatchList로 내려주도록 하겠습니다.

```js
// App.js

import React, { Component } from 'react';
import MatchTemplate from './MatchTemplate/MatchTemplate';
import MatchFinder from './MatchFinder';
import Match from './Match';
import dateFormatter from '../utils/dateFormatter';

class App extends Component {
    state = {
        range: {
            startDate: '2020-01-01',
            endDate: '2020-02-01'
        }
    };

    handleRange = range => {
        const startDate = dateFormatter(range[0]);
        const endDate = dateFormatter(range[1]);

        this.setState({
            range: {
                startDate,
                endDate
            }
        });
    };

    render() {
        return (
            <div>
                <MatchTemplate header={<MatchFinder setRange={this.handleRange}/>}>
                    <Match />
                </MatchTemplate>
            </div>
        );
    }
}

export default App;
```

여기서 dateFormatter는 왜 사용했을까요?

calendar에서 관리되고 있는 state를 보면 배열로 관리되고 있고 날짜도 yyyy-mm-dd형식이 아닙니다.

따라서 api문서에서 원하는 형식으로 데이터를 파싱해줘야 합니다.

데이터 파싱은 미리 준비한 dateFormatter를 통해서 해주세요.

MatchFinder에서는 다시 캘린더로 props를 내려주세요.

```js
import React, { Component } from 'react';
import Calendar from './Calendar';
import League from './League';

class MatchFinder extends Component {
    render() {
        const { setRange } = this.props;

        return (
            <div>
                <League />
                <Calendar setRange={setRange} />
            </div>
        );
    }
}

export default MatchFinder;
```

이렇게 했으면 Calendar.js에서는 button에 onClick이벤트를 등록하고 handleSubmit 이벤트핸들러를 작성해줄게요.

```js
// Calendar.js

import React, { Component } from 'react';
import ReactCalendar from 'react-calendar';
import './Calendar.scss';

class Calendar extends Component {
    state = {
        date: null
    };

    handleChange = date => {
        this.setState({
            date
        });
    };

    handleSubmit = e => {
        e.preventDefault();

        this.props.setRange(this.state.date);
    };

    render() {
        return (
            <div className="calendar">
                <ReactCalendar
                    className="react-calendar"
                    onChange={this.handleChange}
                    selectRange={true}
                />
                <button className="calender-button" onClick={this.handleSubmit}>
                    검색
                </button>
            </div>
        );
    }
}

export default Calendar;
```

리액트 개발자도구를 열고 확인해보세요.

달력에서 날짜를 선택하고 버튼을 누르면 App에 올바른 state값이 잘 담기나요?

이제 App에서 MatchList에 변화된 range값을 내려줄게요.

```js
// App.js

import React, { Component } from 'react';
import MatchTemplate from './MatchTemplate/MatchTemplate';
import MatchFinder from './MatchFinder';
import Match from './Match';
import dateFormatter from '../utils/dateFormatter';

class App extends Component {
    state = {
        range: {
            startDate: '2020-01-01',
            endDate: '2020-02-01'
        }
    };

    handleRange = range => {
        const startDate = dateFormatter(range[0]);
        const endDate = dateFormatter(range[1]);

        this.setState({
            range: {
                startDate,
                endDate
            }
        });
    };

    render() {
        const {range} = this.state;

        return (
            <div>
                <MatchTemplate header={<MatchFinder setRange={this.handleRange}/>}>
                    <Match range={range}/>
                </MatchTemplate>
            </div>
        );
    }
}

export default App;
```

그렇게하면 MatchList에서는 변한 range값에 따른 api요청을 다시 해줘야합니다.

아까는 cdm으로 관리했다면 이번에는 cdu를 사용해볼게요.

```js
// MatchList.js

import React, { Component } from 'react';
import Match from './Match';
import axios from 'axios';

class MatchList extends Component {
    state = {
        loading: false,
        data: null
    };

    getData = async () => {
        this.setState({
            loading: true
        });

        try {
            // 동적으로 쿼리생성
            const { startDate, endDate } = this.props.range;
            const extraQuery = `&from=${startDate}&to=${endDate}`;

            // 템플릿 리터럴 사용
            const response = await axios.get(
                `https://apiv2.apifootball.com/?action=get_events${extraQuery}&league_id=148&APIkey=<본인API_KEY>`
            );

            console.log(response.data);
            this.setState({
                data: response.data
            });
        } catch (e) {
            console.error(e);
        }

        this.setState({
            loading: false
        });
    };

    componentDidMount() {
        this.getData();
    }

    // cdu등록
    componentDidUpdate(prevProps, prevState) {
        if (this.props.range !== prevProps.range) {
            this.getData();
        }
    }

    render() {
        const { loading, data } = this.state;
        // !data.error 조건 추가
        return (
            <div>
                {loading && <h3 style={{ textAlign: 'center' }}>데이터를 불러오는중입니다...</h3>}
                {!loading && data && !data.error &&
                    data.map(d => <Match key={d.match_id} data={d} />)}
            </div>
        );
    }
}

export default MatchList;
```

여기까지하고 이번엔 league종류에따라서 api call을 다시 하게 해볼게요.

우리가 할 작업은 두가지입니다.

바뀐 리그종류에 따라 api call을 다시하기, 그리고 클릭한 메뉴의 스타일을 바꿔주는 것. 이 두가지 입니다.

```js
// App.js

import React, { Component } from 'react';
import MatchTemplate from './MatchTemplate/MatchTemplate';
import MatchFinder from './MatchFinder';
import Match from './Match';
import dateFormatter from '../utils/dateFormatter';

class App extends Component {
    state = {
        //추가
        leagueId: 148,
        range: {
            startDate: '2020-01-01',
            endDate: '2020-02-01'
        }
    };

    handleRange = range => {
        const startDate = dateFormatter(range[0]);
        const endDate = dateFormatter(range[1]);

        this.setState({
            range: {
                startDate,
                endDate
            }
        });
    };

    // 추가
    handleLeagueId = leagueId => {
        this.setState({
            leagueId
        });
    };

    render() {
        const { leagueId, range } = this.state;

        return (
            <div>
                // leagueId는 둘다한테 내려줌
                <MatchTemplate
                    header={
                        <MatchFinder
                            setRange={this.handleRange}
                            setLeagueId={this.handleLeagueId}
                            leagueId={leagueId}
                        />
                    }
                >
                    <Match range={range} leagueId={leagueId} />
                </MatchTemplate>
            </div>
        );
    }
}

export default App;
```

이제 Matchfinder에서 리그로 props를 내려주세요.

```js
// MatchFinder.js

import React, { Component } from 'react';
import Calendar from './Calendar';
import League from './League';

class MatchFinder extends Component {
    render() {
        const { setRange, setLeagueId, leagueId } = this.props;

        return (
            <div>
                <League setLeagueId={setLeagueId} leagueId={leagueId} />
                <Calendar setRange={setRange} />
            </div>
        );
    }
}

export default MatchFinder;
```

```js
// LeagueList.js
class LeagueList extends Component {
    render() {
        const { setLeagueId, leagueId } = this.props;
        return (
            <div>
                {
                    <ul className="leagueList-wrapper">
                        {leagueTypes.map(league => {
                            return (
                                <li key={league.league_id} className="leagueList">
                                    <LeagueItem
                                        league_name={league.league_name}
                                        league_id={league.league_id}
                                        setLeagueId={setLeagueId}
                                        selected={leagueId === league.league_id ? true : false}
                                    />
                                </li>
                            );
                        })}
                    </ul>
                }
            </div>
        );
    }
}

export default LeagueList;
```

```js
// LeagueItem.js

import React, { Component, Fragment } from 'react';
import './LeagueItem.scss';

class LeagueItem extends Component {
    render() {
        const { league_name, league_id, setLeagueId, selected } = this.props;

        return (
            <Fragment>
                <span
                    className={`league ${selected && 'selected'}`}
                    onClick={() => setLeagueId(league_id)}
                >
                    {league_name}
                </span>
            </Fragment>
        );
    }
}

export default LeagueItem;
```

여기까지하고 개발자도구로 확인해보세요. 새로운 리그를 선택하면 leagueId가 App에서 잘 담길 것입니다!

마찬가지로 바뀐 리그에 따른 api호출을 해주면 끝나겠죠?

```js
// MatchList.js

...            
            const { startDate, endDate } = this.props.range;
            const { leagueId } = this.props;
            const extraQuery = `&from=${startDate}&to=${endDate}&league_id=${leagueId}`
...            

    componentDidUpdate(prevProps, prevState) {
        if (
            this.props.range !== prevProps.range ||
            this.props.leagueId !== prevProps.leagueId
        ) {
            this.getData();
        }
    }
```

이제 기능 구현을 다 해보았습니다.

여기에 추가적으로 LeagueItem, 그리고 Match에 shouldComponentUpdate 라이프사이클을 추가해주면 좋습니다.

배열로 관리하는 것들은 기본적으로 scu를 달아주신 다고 생각하시면 편합니다!

