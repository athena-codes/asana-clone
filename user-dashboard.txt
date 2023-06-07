#  USER DASHBOARD GET ROUTE NOTES

from flask import Blueprint, jsonify
from flask_login import login_required, current_user
from app.models import Project, Task

dashboard_routes = Blueprint('dashboard_routes', __name__)

@dashboard_routes.route('/api/user/dashboard', methods=['GET'])
# User must be logged in to access this endpoint
@login_required
def get_user_dashboard():
    """
    Retrieves the user's relevant tasks and projects for the dashboard
    """
    user = current_user

    # User's team + assigned tasks comes from user model
    user_teams = user.user_teams
    assigned_tasks = user.assigned_tasks

    # Dashboard data dictionary that will be populated with user's relevant tasks and project info,
    # as well as the basic info for their profile
    dashboard_data = {
        'id': user.id,
        'firstName': user.firstName,
        'lastName': user.lastName,
        'email': user.email,
        'assigned_tasks': [],
        'projects': []
    }

    # For each assigned task in the assigned_task dict we got from the User model + add that info
    # to the dictionary for our task data
    for task_id, task_data in assigned_tasks.items():
        task_dict = {
            'id': task_id,
            'name': task_data['name'],
            'description': task_data['description'],
            'due_date': task_data['due_date'],
            'completed': task_data['completed'],
            'owner': task_data['owner']['firstName'],
            'assignee': task_data['assignee'],
            'project': task_data['project']['name'],
            'comments': []
        }

        # Comments are a part of task model so query for that data using task_data.get
        #  and for each comment, a dict is created
        comments = task_data.get('comments', {})
        for comment_id, comment_data in comments.items():
            comment_dict = {
                'comment': comment_data['comment'],
                'created_at': comment_data['created_at'],
                'user': comment_data['user']['firstName']
            }
        #  Append the comments to the task dict data now
            task_dict['comments'].append(comment_dict)

        #  Append assigned task data to our overall dashboard data object
        dashboard_data['assigned_tasks'].append(task_dict)

    # Using user_team (part of User model), iterate and find the project's that team ID matched the
    #  teams our current user is apart of
    for user_team in user_teams.values():
        team_id = user_team['team_id']
        projects = Project.query.filter_by(team_id=team_id).all()

        for project in projects:
            project_dict = {
                'id': project.id,
                'name': project.name,
                'description': project.description
            }
            dashboard_data['projects'].append(project_dict)

    # Return the dashboard data as a JSON response
    return jsonify(dashboard_data), 200
