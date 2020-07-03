기존에는 사용자가 웹페이지에 처음 들어가면 componentDidMount에서 풋볼 api를 호출해서 데이터를 보여줬습니다.

그리고 MatchFinder에서 달력에서 새로운 날짜를 선택하거나 헤더메뉴에서 새로운 리그를 선택하면,

그에 맞게 변화된 값을 전부 state로 관리해서 componentDidUpdate를 통해 새로운 api를 호출했습니다.

메뉴같은 간단한 것들은 이렇게 전부 state로 관리해도 상관은 없지만,  어플리케이션의 특성에 따라 얼마든지 다르게 구현할 수 있습니다.

프로젝트2에서 한 것처럼 모든 것을 state로 관리하면 브라우저 히스토리를 사용하지 못합니다.

브라우저 히스토리에 담기지 않으면 사용자가 직접 뒤로가기 앞으로가기를 할 수 없습니다.

반면에 리액트 라우터를 이용하면 히스토리를 사용할 수 있는데요, 이를 이용해서 프로젝트를 조금 바꿔보겠습니다.

**우선 리액트 라우터를 적용해야하므로 리액트라우터를 설치하고 브라우저 라우터를 적용하겠습니다.**

```js
// index.js

import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';
import * as serviceWorker from './serviceWorker';
import './styles/main.scss';
import { BrowserRouter } from 'react-router-dom';

ReactDOM.render(
	<BrowserRouter>
		<App />
	</BrowserRouter>,
	document.getElementById('root')
);
serviceWorker.unregister();
```

**그리고 App에서 기존 코드를 삭제하고 라우트를 설정하고 그 라우트에맞는 컴포넌트를 렌더해주겠습니다.**

기존 App에 있던 코드는 src/pages/Matchpage.js를 만들어서 옮겨주도록 할게요!

```js
// src/App.js

import React, { Component } from 'react';
import { Route, Switch } from 'react-router-dom';
import MatchPage from '../pages/MatchPage';

class App extends Component {
	render() {
		return (
			<div>
				<Switch>
					<Route path="/match/:leagueName" component={MatchPage} />
					<Route render={() => <div>wrong url</div>} />
				</Switch>
			</div>
		);
	}
}

export default App;

```

```js
// src/pages/MatchPage.js

import React, { Component } from 'react';
import MatchTemplate from '../components/MatchTemplate';
import MatchFinder from '../components/MatchFinder';
import Match from '../components/Match';

class MatchPage extends Component {
	render() {
		return (
			<MatchTemplate header={<MatchFinder />}>
				<Match />
			</MatchTemplate>
		);
	}
}

export default MatchPage;

```

이제 localhost:3000/match/premier?from=2020-01-01&to=2020-02-01 이런식으로 접속했을때 파라미터와 쿼리를 잘 가져와서 자식 컴포넌트에게 내려주겠습니다.

```js
// src/pages/MatchPage.js


import React, { Component } from 'react';
import qs from 'qs';
import MatchTemplate from '../components/MatchTemplate';
import MatchFinder from '../components/MatchFinder';
import Match from '../components/Match';

class MatchPage extends Component {
	render() {
		const { match, location } = this.props;
		
		const { leagueName } = match.params;
		const range = qs.parse(location.search.substr(1));

		return (
			<MatchTemplate header={<MatchFinder />}>
				<Match range={range} leagueName={leagueName} />
			</MatchTemplate>
		);
	}
}

export default MatchPage;
```

기존의 MatchList를 다음처럼 수정해주세요.

```js
import React, { Component } from 'react';
import axios from 'axios';
import Match from './Match';
import { leagueTypes } from '../MatchFinder/League/LeagueList';

class MatchList extends Component {
	state = {
		loading: false,
		data: null,
	};

	getData = async () => {
		this.setState({
			loading: true,
		});

		try {
			const { from, to } = this.props.range;
			const { leagueName } = this.props;
			const { league_id } = leagueTypes.find((l) => l.league === leagueName);

			const { data } = await axios.get(
				`https://apiv2.apifootball.com/?action=get_events&from=${from}&to=${to}&league_id=${league_id}&APIkey=발급받은apikey`
			);

			this.setState({
				data,
			});
		} catch (e) {
			console.error(e);
		}

		this.setState({
			loading: false,
		});
	};

	componentDidMount() {
		this.getData();
	}

	componentDidUpdate(prevProps, prevState) {
		if (
			this.props.range !== prevProps.range ||
			this.props.leagueId !== prevProps.leagueId
		) {
			this.getData();
		}
	}

	render() {
		const { loading, data } = this.state;

		return (
			<div>
				{loading && (
					<h1 style={{ textAlign: 'center' }}>데이터를 불러오는 중입니다...</h1>
				)}
				{!loading &&
					data &&
					!data.error &&
					data.map((d) => {
						return <Match key={d.match_id} data={d} />;
					})}
			</div>
		);
	}
}

export default MatchList;

```

import { leagueTypes } from '../MatchFinder/League/LeagueList'; 를 하기위해서는 LeagueList.js에서 leagueTypes를 export해주어야하고, leagueTypes안에 premier는 148번, laliga는 468번 이런식으로 매핑을 해줘야하므로 새로운 값들을 추가해주겠습니다.

```js
// LeagueList.js

...

export const leagueTypes = [
	{
		league: 'premier',
		league_name: '프리미어 리그',
		league_id: 148,
	},
	{
		league: 'laliga',
		league_name: '라리가',
		league_id: 468,
	},
	{
		league: 'serie',
		league_name: '세리에 A',
		league_id: 262,
	},
	{
		league: 'bundes',
		league_name: '분데스리가',
		league_id: 196,
	},
	{
		league: 'legue',
		league_name: '리그 앙',
		league_id: 176,
	},
	{
		league: 'champs',
		league_name: '챔피언스리그',
		league_id: 149,
	},
];

...
```

이제 localhost:3000/match/premier?from=2020-01-01&to=2020-02-01으로 접속하면 잘 작동할 것입니다.

다음으로 달력에서 날짜를 바꾸고 확인버튼을 눌렀을때 url이 변하면서 다른 페이지로 이동하는 것을 구현해보겠습니다.

```js
import React, { Component } from 'react';
import ReactCalendar from 'react-calendar';
import './Calendar.scss';
import dateFormatter from '../../../utils/dateFormatter';
import { withRouter } from 'react-router-dom';

class Calendar extends Component {
	state = {
		date: null,
	};

	handleChange = (date) => {
		this.setState({
			date,
		});
	};

	handleSubmit = (e) => {
		e.preventDefault();

		const { date } = this.state;
		if (date === null) return;
		const from = dateFormatter(date[0]);
		const to = dateFormatter(date[1]);
		const { history, match } = this.props;

		history.push(`${match.url}?from=${from}&to=${to}`);
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

export default withRouter(Calendar);
```

여기서는 withRouter라는걸 사용했는데요, 만약 &lt;Route path="어쩌고저쩌고' component={Calendar} /&gt; 이런식으로 Calendar컴포넌트 자체가 Route에 의해 렌더되었더라면 history, match, location을 사용할 수 있었을 것입니다. 하지만 Calendar컴포넌트는 그냥 MatchFinder가 렌더해주고 있으므로 위세가지 props들이 없는데요, 이럴때 withRouter를 사용하면 됩니다.

이제 다시 개발서버를 열고 날짜를 바꾼뒤 url을 이동해보세요.

이번에는 사용자가 localhost:3000/ 으로 접속했을때 기본 url인 http://localhost:3000/match/premier?from=2020-01-01&to=2020-02-01 로 리다이렉션을 시키겠습니다.

이를 위해 리다이렉션 전용컴포넌트를 만들고, 연결해주도록 할게요.

```js
// src/common/Redirection.js

import React, { Component } from 'react';
import { Redirect } from 'react-router-dom';

class Redirection extends Component {
	render() {
		const { pathname } = this.props.location;

		return (
			<div>
				{pathname === '/' && (
					<Redirect to="/match/premier?from=2020-02-01&to=2020-03-01" />
				)}
			</div>
		);
	}
}

export default Redirection;
```

App에 새로운 경로를 추가해주겠습니다.

```js
import React, { Component } from 'react';
import { Switch, Route } from 'react-router-dom';
import MatchPage from '../pages/MatchPage';
import Redirection from '../common/Redirection';

class App extends Component {
	render() {
		return (
			<Switch>
				<Route exact path="/" component={Redirection} />
				<Route path="/match/:leagueName" component={MatchPage} />
				<Route render={() => <div>wrong url</div>} />
			</Switch>
		);
	}
}

export default App;
```

이렇게 Redirection컴포넌트를 만나면 to로 지정해둔 주소로 리디렉션이 됩니다. history.push와 다른점은 리디렉션 되기 전 경로는 브라우저 히스토리에 담기지 않는 다는 것입니다.

리그아이템의 경로는 NavLink를 사용해서 구현할 수 있습니다. NavLink를 사용하면 사용자가 클릭했을때~의 selected와 같은 것을 구현하지 않아도 되기때문에 아주 간단하게 구현할 수 있습니다. 

여기까지 리팩토링을 해보았습니다.

