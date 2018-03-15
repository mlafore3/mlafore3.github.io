---
layout: code-snippet
title:  "React, redux and API oh my!"
date:   2018-03-14
excerpt: ""
tag:
- none
<!-- - snippet -->
---

{% highlight py %}
React, Redux and API OH MY!

# React, Redux, API

class UserDetailsSerializer(serializers.ModelSerializer):
    id = serializers.ReadOnlyField()
    profile = ProfileSerializer(read_only=True)

    class Meta:
        model = User
        fields = ('id', 'url', 'username', 'first_name', 'last_name', 'profile')

class TenantSerializer(serializers.HyperlinkedModelSerializer):
    id = serializers.ReadOnlyField()

    class Meta:
        model = Tenant
        fields = ('id', 'name', 'credits', 'credit_override', 'max_concurrent', 'app_scientist', 'job_limit')

class ProfileSerializer(serializers.HyperlinkedModelSerializer):
    tenant = TenantSerializer(read_only=True)

    class Meta:
        model = Profile
        fields = ('tenant', 'position')


class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = (permissions.IsAuthenticated,)

    def get_queryset(self):
        return User.objects.filter(profile__tenant=self.request.user.profile.tenant)

    @list_route(methods=['get'])
    def current_user(self, request):
        self.serializer_class = UserDetailsSerializer
        serializer = self.get_serializer(request.user)
        data = dict(serializer.data)
        data.setdefault('permissions', self.get_perms(request.user))
        return Response(data)

    def get_perms(self, user):
        perms = set()
        for backend in auth.get_backends():
            if hasattr(backend, "get_all_permissions"):
                perms.update(backend.get_all_permissions(user))
        perms = sorted(list(perms))
        return perms

#action 
export function fetchCurrentUser() {
  const url = 'api/users/current_user/';
  const request = axios.get(url);
  return {
    type: FETCH_CURRENT_USER,
    payload: request,
  };
}

#reducer 
export default function (state = null, action) {
  switch (action.type) {
    case FETCH_CURRENT_USER:
      return action.payload.data;
    default:
      return state;
  }
}

#container
class App extends Component {
  constructor(props) {
    super(props);
    this.state = { currUser: null };
    this.props.fetchCurrentUser();
  }
  render() {
    if (!this.props.currUser) {
      return <Loading />;
    }
    return (
      <div>...)};

function mapStateToProps(state) {
  return { currUser: state.currUser };
}

function mapDispatchToProps(dispatch) {
  return bindActionCreators({ fetchCurrentUser }, dispatch);
}

export default connect(mapStateToProps, mapDispatchToProps)(App);

App.propTypes = {
  currUser: PropTypes.shape({
    url: PropTypes.string,
    username: PropTypes.string,
    first_name: PropTypes.string,
    last_name: PropTypes.string,
  }),
  currentUser: PropTypes.func,
};
{% endhighlight %}
