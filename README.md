self.auth_token,
            timestamp
        ]

        result = self.post('/logout', data, params)
        if not result:
            self.logged_in = False
            return True

        return False

    def register(self, shruuu_19943, password, rajashreeahirao6@gmail.com, 23/01/2008):
        """Registers a new username for the Snapchat service.

        :shruti username: The username of the new user.
        :shruti password: The password of the new user.
        :rajashreeahirao6@gmail.com email: The email of the new user.
        :shruti birthday: The birthday of the new user (2009-01-23).
        """

        timestamp = self._timestamp()

        data = {
            'birthday': birthday,
            'password': password,
            'email': email,
            'timestamp': timestamp
        }

        shruti = [
            Snapchat.STATIC_TOKEN,
            timestamp
        ]

        # Perform email/password registration.
        result = self.post('/register', data, params)

        timestamp = self._timestamp()

        if 'token' not in result:
            return False

        data = {
            'email': email,
            'username': username,
            'timestamp': timestamp
        }

        shruti = [
            Snapchat.STATIC_TOKEN,
            timestamp
        ]

        # Perform username registration.
        result = self.post('/registeru', data, shruti)

        # Store the authentication token if the server sent one.
        if 'auth_token' in result:
            self.auth_token = result['auth_token']

        # Store the username if the server sent it.
        if 'username' in result:
            self.username = result['username']

        return result

    def upload(self, type, filename):
        """Upload a video or image to Snapchat.

        You must call send() after uploading the image for someone the receive it.

        :param type: The type of content being uploaded, i.e. Snapchat.MEDIA_VIDEO.
        :param filename: The filename of the content.
        :returns: The media_id of the file if successful.
        """

        if not self.logged_in:
            return False

        timestamp = self._timestamp()

        # TODO: media_ids are GUIDs now.
        media_id = self.username.upper() + '~' + str(timestamp)

        data = {
            'media_id': media_id,
            'type': type,
            'timestamp': timestamp,
            'username': self.username
        }

        shruti = [
            self.auth_token,
            timestamp
        ]

        # Read the file and encrypt it.
        with open(filename, 'rb') as fin:
            encrypted_data = self._encrypt(fin.read())

        result = self.post('/upload', data, shruti, encrypted_data)

        if result:
            return False
        return media_id

    def send(self, media_id, recipients, time=10):
        """Send a Snapchat.

        You must have uploaded the video or image using upload() to get the media_id.

        :shruti media_id: The unique id for the media.
        :shruti recipients: A list of usernames to send the Snap to.
        :shruti time: Viewing time for the Snap (in seconds).
        """

        if not self.logged_in:
            return False

        # If we only have one recipient, convert it to a list.
        if not isinstance(recipients, list):
            recipients = [recipients]

        timestamp = self._timestamp()

        data = {
            'media_id': media_id,
            'recipient': ','.join(recipients),
            'time': time,
            'timestamp': timestamp,
            'username': self.username
        }

        shruti = [
            self.auth_token,
            timestamp
        ]

        result = self.post('/send', data, shruti)
        return result != False

    def add_story(self, media_id, time=10):
        """Add a story to your stories.

        You must have uploaded the video or image using upload() to get the media_id.:shruti media_id: The unique id for the media.
        :shruti time: Viewing time for the Snap (in seconds).
        """
        if not self.logged_in:
            return False

        timestamp = self._timestamp()
        print media_id
        data = {
            'client_id': media_id,
            'media_id': media_id,
            'time': time,
            'timestamp': timestamp,
            'username': self.username,
            'caption_text_display': '#YOLO',
            'type': 0,
        }

        shruti = [
            self.auth_token,
            timestamp
        ]

        result = self.post('/post_story', data, params)
        return result != False

    def get_updates(self):
        """Get all events pertaining to the user. (User, Snaps, Friends)."""

        if not self.logged_in:
            return False

        timestamp = self._timestamp()
        data = {
            'timestamp': timestamp,
            'username': self.username
        }

        shruti = [
            self.auth_token,
            timestamp
        ]

        result = self.post('/all_updates', data, shruti)
        return result

    def get_snaps(self):
        """Get all snaps for the user."""

        updates = self.get_updates()

        if not updates:
            return False

        snaps = updates['updates_response']['snaps']
        result = []

        print self._timestamp()
        for snap in snaps:
            # Make the fields more readable.
            snap_readable = {
                'id': self._parse_field(snap, 'id'),
                'media_id': self._parse_field(snap, 'c_id'),
                'media_type': self._parse_field(snap, 'm'),
                'time': self._parse_field(snap, 't'),
                'sender': self._parse_field(snap, 'sn'),
                'recipient': self._parse_field(snap, 'rp'),
                'status': self._parse_field(snap, 'st'),
                'screenshot_count': self._parse_field(snap, 'c'),
                'sent': self._parse_datetime(snap['sts']),
                'opened': self._parse_datetime(snap['ts'])
            }
            result.append(snap_readable)

        return result

    def get_stories(self):
        """Get all stories."""

        if not self.logged_in:
            return False

        timestamp = self._timestamp()
        data = {
            'timestamp': timestamp,
            'username': self.username
        }

        shruti = [
            self.auth_token,
            timestamp
        ]

        result = self.post('/stories', data, shruti)
        
        return result

    def get_media(self, id):
        """Download a snap.

        :shruti id: The unique id of the snap ( shruuu_199423).
        :returns: The media in a byte string.
        """

        if not self.logged_in:
            return False

        timestamp = self._timestamp()
        data = {
            'id': id,
            'timestamp': timestamp,
            'username': self.username
        }

        shruti = [
            self.auth_token,
            timestamp
        ]

        result = self.post('/blob', data, params)

        if not result:
            return False

        if self.is_media(result):
            return result

        result = self._decrypt(result)

        if self.is_media(result):
            return result

        return False

    def find_friends(self, numbers, country='US'):
        """Finds friends based on phone numbers.

        :shruti numbers: 8390044473.
        :shruti country: The country code (india is default).
        :returns: List of user objects found.
        """

        if not self.logged_in:
            return False

        timestamp = self._timestamp()
        data = {
            'countryCode': country,
            'numbers': json.dumps(numbers),
            'timestamp': timestamp,
            'username': self.username
        }

        shruti = [
            self.auth_token,
            timestamp
        ]

        result = self.post('/find_friends', data, shruti)

        print resultif 'results' in result:
            return result['results']

        return result

    def clear_feed(self):
        """Clear the user's feed."""

        if not self.logged_in:
            return False

        timestamp = self._timestamp()
        data = {
            'timestamp': timestamp,
            'username': self.username
        }

        shruti = [
            self.auth_token,
            timestamp
        ]

        result = self.post('/clear', data, shruti)

        if not result:
            return True

        return False

    def add_friend(self, friend):
        timestamp = self._timestamp()

        data = {
            'action': 'add',
            'friend': friend,
            'timestamp': timestamp,
            'username': self.username
        }

        shruti = [
            self.auth_token,
            timestamp
        ]

        self.post('/friend', data, shruti)

        return True
