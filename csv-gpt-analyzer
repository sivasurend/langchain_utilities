import streamlit as st
import pandas as pd
from langchain.agents import create_csv_agent
from langchain.llms import OpenAI
import boto3
import uuid
from io import StringIO
from tabulate import tabulate

def main():
    st.title("CSV-GPT")
    st.write("Analyze your CSV file instantly")

    if 'uploaded' not in st.session_state:
        st.session_state.uploaded = False
        st.session_state.s3_file_path = None

    uploaded_file = st.file_uploader("Upload a CSV file", type=['csv'])

    bucket_name = 'csv-gpt-uploads'

    s3 = boto3.client(
        's3',
        aws_access_key_id=st.secrets.aws_access_key_id,
        aws_secret_access_key=st.secrets.aws_secret_access_key
    )

    if uploaded_file is not None and not st.session_state.uploaded:
        df = pd.read_csv(uploaded_file)
        file_name = f"{uuid.uuid4().hex}.csv"

        csv_buffer = StringIO()
        df.to_csv(csv_buffer)

        s3.put_object(Body=csv_buffer.getvalue(), Bucket=bucket_name, Key=file_name)

        st.session_state.uploaded = True
        st.session_state.s3_file_path = f"https://{bucket_name}.s3.amazonaws.com/{file_name}"

        st.success('File Uploaded Successfully! Now Start Analyzing.')

    analysis_query = st.text_input("Enter the analysis query")

    if st.button("Analyze Now"):
        if st.session_state.uploaded and st.session_state.s3_file_path is not None:
            agent = create_csv_agent(OpenAI(temperature=0, openai_api_key=st.secrets.openai_api_key),
                                     st.session_state.s3_file_path,
                                     Model='gpt-4'
                                     )
            result = agent.run(analysis_query)

            # Display the output
            st.write(result)
        else:
            st.warning("Please upload a CSV file first.")

# Run the app
if __name__ == "__main__":
    main()
