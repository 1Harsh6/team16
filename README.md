# team16
import openpyxl
import google.auth
from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from googleapiclient.discovery import build
from linkedin_api import Linkedin
import facebook


# Function to authenticate Google credentials
def authenticate_google_credentials():
    credentials = None
    if os.path.exists('token.json'):
        credentials = Credentials.from_authorized_user_file('token.json', SCOPES)

    if not credentials or not credentials.valid:
        if credentials and credentials.expired and credentials.refresh_token:
            credentials.refresh(Request())
        else:
            flow = google.auth.OAuth2FlowFromClientSecrets(CLIENT_SECRETS_FILE, SCOPES)
            credentials = flow.run_local_server(port=0)

        with open('token.json', 'w') as token:
            token.write(credentials.to_json())

    return credentials


# Function to upload video to YouTube
def upload_video_to_youtube(youtube, video_file, title, description, tags, category_id):
    request_body = {
        'snippet': {
            'title': title,
            'description': description,
            'tags': tags,
            'categoryId': category_id
        },
        'status': {
            'privacyStatus': 'public'
        }
    }

    media_file = MediaFileUpload(video_file)

    response_upload = youtube.videos().insert(
        part='snippet,status',
        body=request_body,
        media_body=media_file
    ).execute()

    return response_upload['id']


# Function to upload post to LinkedIn
def upload_post_to_linkedin(linkedin_api, message, image_file):
    image_data = open(image_file, 'rb').read() if image_file else None
    response = linkedin_api.submit_share(
        text=message,
        image_data=image_data
    )

    return response


# Function to upload post to Facebook
def upload_post_to_facebook(graph_api, message, image_file):
    image_data = open(image_file, 'rb').read() if image_file else None
    response = graph_api.put_photo(image_data, message=message)

    return response


# Load the workbook
workbook = openpyxl.load_workbook('schedule.xlsx')

# Select the worksheet
worksheet = workbook['Sheet1']

# Authenticate Google credentials for YouTube API
CLIENT_SECRETS_FILE = 'client_secret.json'
SCOPES = ['https://www.googleapis.com/auth/youtube.upload']
youtube = build('youtube', 'v3', credentials=authenticate_google_credentials())

# Authenticate LinkedIn credentials
LINKEDIN_USERNAME = 'your linkedin username'
LINKEDIN_PASSWORD = 'your linkedin password'
linkedin_api = Linkedin(LINKEDIN_USERNAME, LINKEDIN_PASSWORD)

# Authenticate Facebook credentials
graph_api = facebook.GraphAPI(FACEBOOK_ACCESS_TOKEN)

# Iterate through the rows of the worksheet
for row in worksheet.iter_rows(min_row=2, values_only=True):
    platform = row[0]
    title = row[1]
    description = row[2]
    tags = row[3]
    category_id = row[4]
    video_file = row[5]
    image_file = row[6]
    message = row[7]

    if platform == 'YouTube':
        # Upload video to YouTube
        video_id = upload_video_to_youtube(youtube, video_file, title, description, tags, category_id)
        print(f"Video uploaded to YouTube with ID: {video_id}")
